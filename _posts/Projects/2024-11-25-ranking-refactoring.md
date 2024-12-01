---
title: "[Refactoring] 첫 코드 리팩토링 기록"
excerpt: "-"

writer: Hwayeon Hong
# categories: projects
# permalink: /projects/:title/
tags:
  - [java]

toc: true
toc_sticky: true

date: 2024-11-25
last_modified_at: 2024-11-25
---

> 이전 시간에서 랭킹 차트를 구현하면서 쿼리문을 최적화하는 방법을 고민해 보았다. 그렇게 해서 완성된 코드는 각 과목별로 first score, last score 를 각각 두 번 요청하는 방식으로 구성되었다. NCA 를 만들 때 까지는 나쁘지 않아 보였는데... NCP 구현에 들어가면서 Axios 요청을 좀 더 줄일 수 있지 않나? 하는 생각이 들었다. 웹 사이트를 최적화 함에 있어서 HTTP 요청 최소화는 기본 중의 기본이다. 자 지금부터 NCA 때 구현했던 코드를 리팩토링 해보자 !

### 1️⃣ 기존 코드 분석

🔯 FrontEnd

아래 코드가 문제의 Axios 호출 부분이다. getRankingData 메서드 내부에서 if 문으로 tableName 에 따라 Axios 호출을 분기하고 있는 것처럼 보이지만, 실상은 렌더링 될 때 마다 두 테이블에서 데이터를 전부 가져와야 한다. 다시 말해 분기할 필요가 전혀 없는 로직이다...

```javascript
const [firstData, setFirstData] = useState([]);
const [lastData, setLastData] = useState([]);
const table = ["first_score", "last_score"];

useEffect(() => {
  if (firstData.length === 0) {
    getRankingData(table[0]);
    getRankingData(table[1]);
  }
}, []);

const getRankingData = async (tableName) => {
  if (tableName === "first_score") {
    const response = await axios
      .post(`/ranking/v2`, {
        title: "NCA",
        table: tableName,
      })
      .catch((err) => {
        console.log(err);
      });

    setFirstData(response.data);
  } else {
    const response = await axios
      .post(`/ranking/v2`, {
        title: "NCA",
        table: tableName,
      })
      .catch((err) => {
        console.log(err);
      });

    setLastData(response.data);
  }
};
```

☕️ BackEnd

단순한 Read 작업으로, 프론트에서 title 과 table 정보를 받아서 조회하는 코드

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
    return rankMapper.findTop5V2(rankDTO);
}
```

### 2️⃣ 리팩토링

리팩토링 작업은 생각 외로 간단했다. 프론트에서는 중복 코드를 삭제했고, 백에서는 필요한 데이터를 한 번에 리턴하도록 바꿔주었다.

추가로, NCA 와 달리 다른 과목들은 '과목 코드' 라는 것이 있어서 과목별로 랭킹을 집계해야 한다. 이 부분도 한 번의 통신으로 모든 데이터를 가져올 수 있도록 리팩토링 했다.

🔯 FrontEnd

- 프론트에서는 랭킹을 조회할 과목이 무엇인지만 전달한다. (NCA or NCP)

```javascript
useEffect(() => {
  if (rankingData.length === 0) {
    getRankingData();
  }
}, []);

const getRankingData = async () => {
  const response = await axios
    .post(`/ranking/v2`, {
      title: "NCA",
    })
    .catch((err) => {
      console.log(err);
    });

  console.log(response.data);
  setRankingData(response.data);
};
```

☕️ BackEnd

- 프론트에서 요청이 들어오면 first_score, last_score 두 개의 테이블에서 랭킹 데이터를 조회하는 로직을 수행한다.

- NCP 의 경우, '과목 코드' 별로 first_score, last_score 를 조회해야 하므로, 서비스 단계에서 이를 분기했다.

- 추가로, 데이터를 조회하는 로직을 private 메서드로 따로 정의하여 코드를 간결하게 정리했다.

```java
// 컨트롤러
@PostMapping("/ranking/v2")
public ResponseEntity<Map<String, List>> getRankingV2(@RequestBody RankDTO rankDTO) {
    if(rankDTO == null || rankDTO.getTitle() == null) {
        throw new CustomException(
                "title 정보는 필수입니다.",
                "INVALID_RANK_INFO",
                HttpStatus.BAD_REQUEST
        );
    }
    return ResponseEntity.ok(rankService.getRankingV2(rankDTO));
}

// 서비스
@Service
@RequiredArgsConstructor
@Slf4j
public class RankService {

    private final RankMapper rankMapper;
    private Map<String, List> results = new ConcurrentHashMap<>();

    /**
     * 랭킹 조회 메서드
     */
    private void findRanker(RankDTO rankDTO) {
        // 1회차 랭킹 조회
        rankDTO.setTable("first_score");
        List<RankDTO> firstDTO = rankMapper.findTop5V2(rankDTO);

        // 다회차 랭킹 조회
        rankDTO.setTable("last_score");
        List<RankDTO> lastDTO = rankMapper.findTop5V2(rankDTO);

        // 리턴값 세팅
        String first = rankDTO.getTitle() + "first"; // EX) NCAfirst, NCP200first
        String last = rankDTO.getTitle() + "last"; // EX) NCAlast, NCP200last
        results.put(first, firstDTO);
        results.put(last, lastDTO);
    }

    /**
     * V2 : join 쿼리로 한번에 검색
     */
    public Map<String, List> getRankingV2(RankDTO rankDTO) {
        String title = rankDTO.getTitle();
        log.info("Ranking title: {}", title);

        if(title.equals("NCA")) {
            findRanker(rankDTO);
        }

        if(title.equals("NCP")) {
            // NCP200
            rankDTO.setTitle("NCP200");
            findRanker(rankDTO);

            // NCP202
            rankDTO.setTitle("NCP202");
            findRanker(rankDTO);

            // NCP207
            rankDTO.setTitle("NCP207");
            findRanker(rankDTO);
        }

        return results;
    }

}

}
```

### 🤓 느낀 점

- 과거의 나는 바보였나? 어떻게든 화면을 구현하는 것에만 급급하다 보니 일단 화면에 잘 보이기만 하면 자잘한 것은 넘어가 버리는 안일주의 부실공사가 불편한 코드를 만든 것 같다.

- 최적화를 고민하자 ! 개발을 하면서 굳어지는 생각 하나. 단순히 코드만 짤 수 있다면 코더와 다를 바 없다. 효율, 성능 측면에서 더 나은 로직을 구현할 수 없을까 고민하는 과정이 즐겁다.
