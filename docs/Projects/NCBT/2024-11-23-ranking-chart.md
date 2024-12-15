---
title: "[MySQL] 조인으로 쿼리문 최적화하기"
parent: NCBT
nav_order: 4

tags:
  - [mysql, join]

toc: true
toc_sticky: true

date: 2024-11-23
last_modified_at: 2024-11-23
---

> chart.js 로 랭킹 차트를 구현하면서 쿼리문 최적화에 대한 의문이 생겼다. 우리 서비스에서 랭킹 차트란, 해당 수험 과목을 풀었던 유저들의 기록을 남겨서 순위를 매기는 차트인데, 데이터를 디비에서 적절히 가져오기만 하면 된다. 여기서 '적절히' 가져오려면 어떤 방법을 써야 할까?

### 🔥 테이블 구조 확인

<img width="765" alt="스크린샷 2024-10-23 오전 10 55 29" src="https://github.com/user-attachments/assets/b9eb4a76-1a96-4cfd-ae82-c1b954cb5a49">

- 내가 필요한 데이터 : score, nickname
- 내가 조작해야 하는 테이블 : subject, first_score (last_score), user

위 정보를 토대로 생각해보면, 조인 쿼리 하나를 사용하는 것이 나을지 혹은 여러 개의 쿼리를 실행시키는 것이 나을지 고민이 된다... 그래서 이 참에 직접 테스트를 해보기로 했다. 테스트용으로 10만개의 데이터를 각 테이블에 넣어두었다.

```sql
-- 임시 프로시저를 생성하여 대량의 테스트 데이터 생성
DELIMITER //
CREATE PROCEDURE GenerateTestUsers()
BEGIN
    DECLARE i INT DEFAULT 1;

    -- 트랜잭션 시작
    START TRANSACTION;

    WHILE i <= 100000 DO
        INSERT INTO `user` (
            `email`,
            `nickname`,
            `password`,
            `roles`
        )
        VALUES (
            CONCAT('user', i, '@ncbt.com'),
            CONCAT('User', i),
            '1234',
            'USER'
        );

        SET i = i + 1;

    END WHILE;

    COMMIT;
END //
DELIMITER ;

DELIMITER //
CREATE PROCEDURE GenerateLastScores()
BEGIN
    DECLARE i INT DEFAULT 1;

    -- 트랜잭션 시작
    START TRANSACTION;

    WHILE i <= 100000 DO
        INSERT INTO `last_score` (
            `score`,
            `user_id`,
            `subject_id`
        )
        VALUES (
            -- 0부터 60 사이의 랜덤 점수 생성
            FLOOR(RAND() * 61),
            -- 1부터 999 사이의 랜덤 사용자 ID 선택
            FLOOR(1 + RAND() * 999),
            -- 1부터 4 사이의 랜덤 과목 ID 선택
            FLOOR(1 + RAND() * 4)
        );

        SET i = i + 1;

        -- 1000개 단위로 커밋하여 메모리 관리
        IF i % 1000 = 0 THEN
            COMMIT;
            START TRANSACTION;
        END IF;
    END WHILE;

    -- 마지막 트랜잭션 커밋
    COMMIT;
END //
DELIMITER ;
```

### 1️⃣ MySQL Workbench 의 Duration Time 으로 테스트

워크벤치에서 쿼리를 실행했을 때의 경과 시간을 알아보자. 최초로 단 한 번 실행한 결과를 기록했다.

1. 여러 개의 쿼리를 실행했을 때 측정한 시간 : 약 0.12 sec

```sql
-- 1. 프론트에서 넘겨주는 'title'로 subject_id를 검색 (0.016 sec)
select id from subject where title = 'NCA';

-- 2. 해당하는 subject_id 중에서 score 가 높은 순서로 정렬. (0.104 sec)
select u.nickname, t.score
from user u
inner join first_score t on u.id = t.user_id
where subject_id = 1
order by score desc limit 5;
```

2. 한 개의 조인 쿼리를 실행했을 때 측정한 시간 : 약 0.028 sec

```sql
-- 4. 위 과정을 조인 쿼리로 한 번에 수행할 수 없을까? (0.028 sec)
select u.nickname, f.score
from user u
inner join first_score f on u.id = f.user_id
where f.subject_id = (
select id
from subject
where title = 'NCA'
)
order by f.score desc limit 5;
```

| 여러 개의 쿼리 | 한 개의 조인 쿼리 |
| -------------- | ----------------- |
| 0.12 sec       | 0.028 sec         |

3. 워크벤치에서만 돌려봐도 조인 쿼리를 실행할 때의 실행 시간이 빠른 것을 확인할 수 있다. 사실 워크벤치에서 바로 돌려서 그렇지, 이걸 클라이언트와 통신한다고 생각하면... 쿼리 하나하나 보내는 것 보다 하나의 조인 쿼리를 보내는 게 코드 복잡성도 줄이고 통신 과정에서의 time lose 도 줄일 수 있을 것이라는 생각이 든다.

### 2️⃣ POSTMAN 테스트

잠정적인 결론은 그렇지만, 뭐든지 직접 해봐야 공부도 되고 기억에도 잘 남는 법! 각각의 API 를 만들어서 통신 테스트를 해보기로 했다.

1. 여러 개의 쿼리를 실행했을 때 로직 (v1)

```java
// 컨트롤러
@GetMapping("/ranking/v1")
public ResponseEntity<List<RankDTO>> getRankingV1(@RequestBody RankDTO rankDTO) {
    if(rankDTO == null || rankDTO.getTitle() == null || rankDTO.getTable() == null) {
        throw new CustomException(
                "title 및 table 정보는 필수입니다.",
                "INVALID_RANK_INFO",
                HttpStatus.BAD_REQUEST
        );
    }
    return ResponseEntity.ok(rankService.getRankingV1(rankDTO));
}

// 서비스
public List<RankDTO> getRankingV1(RankDTO rankDTO) {
    // 1. title 로 subject_id 검색
    rankDTO.setSubjectId(rankMapper.findIdByTitle(rankDTO.getTitle()));

    // 2. 해당하는 subject_id 중에서 score 가 높은 순서로 정렬. (5행까지만 추출)
    return rankMapper.findTop5(rankDTO);
}

// 매퍼
@Select("SELECT id FROM subject WHERE title = #{title}")
int findIdByTitle(String title);

@Select("SELECT u.nickname, t.score " +
        "from user u " +
        "inner join ${table} t ON u.id = t.user_id " +
        "where t.subject_id = #{subjectId} " +
        "ORDER BY t.score DESC LIMIT 5")
List<RankDTO> findTop5(RankDTO rankDTO);
```

2. 한 개의 조인 쿼리를 실행했을 때 로직 (v2)

```java
// 컨트롤러
@GetMapping("/ranking/v2")
public ResponseEntity<List<RankDTO>> getRankingV2(@RequestBody RankDTO rankDTO) {
    if(rankDTO == null || rankDTO.getTitle() == null || rankDTO.getTable() == null) {
        throw new CustomException(
                "title 및 table 정보는 필수입니다.",
                "INVALID_RANK_INFO",
                HttpStatus.BAD_REQUEST
        );
    }
    return ResponseEntity.ok(rankService.getRankingV2(rankDTO));
}

// 서비스
public List<RankDTO> getRankingV2(RankDTO rankDTO) {
    // 조인 쿼리로 한 번에 검색
    return rankMapper.findTop5V2(rankDTO);
}

// 매퍼
@Select("SELECT u.nickname, t.score " +
        "FROM user u " +
        "INNER JOIN ${table} t ON u.id = t.user_id " +
        "WHERE t.user_id = " +
        "(SELECT id FROM subject WHERE title = #{title}) " +
        "ORDER BY t.score DESC LIMIT 5")
List<RankDTO> findTop5V2(RankDTO rankDTO);
```

3. POSTMAN 테스트 시작 : 1초마다 50개의 통신을 보내도록 하고 평균값을 내봤다.

#### v1 결과

<img width="1057" alt="스크린샷 2024-10-23 오후 2 28 31" src="https://github.com/user-attachments/assets/319ba6bf-bfa0-43ff-b58c-80f5eb0aba76">

#### v2 결과

<img width="1061" alt="스크린샷 2024-10-23 오후 2 31 47" src="https://github.com/user-attachments/assets/7447999e-d624-4ec2-9f1c-453fdb068199">

| 여러 개의 쿼리 (v1) | 한 개의 조인 쿼리 (v2) |
| ------------------- | ---------------------- |
| 0.069 sec           | 0.022 sec              |

예상했던대로 한 개의 조인 쿼리를 사용하는 v2 로직이 응답 시간이 훨씬 단축되는 것을 확인할 수 있었다. 조인을 적절히 사용하면 여러 번 네트워크를 타는 것 보다 효율적인 결과를 얻을 수 있음을 알게 되었다.