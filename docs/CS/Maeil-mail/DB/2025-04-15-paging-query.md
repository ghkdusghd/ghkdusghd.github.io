---
title: RDB에서 페이징 쿼리의 필요성을 설명해 주세요.
parent: 데이터베이스
nav_order: -5
---

<div style="text-align: center; display: flex;
    flex-direction: column;
    align-items: center;">
    <h5>오늘의 질문</h5>
    <div style="color: black; background-color: #F0F3F5; border-radius: 5px; width: 50%; padding: 20px;">
    RDB에서 페이징 쿼리의 필요성을 설명해 주세요.
    </div>
</div>

---

### 페이징 쿼리 (Paging Query)

전체 데이터를 부분적으로 나눠 데이터를 조회하거나 처리할 때 사용한다. 데이터를 상대적으로 작은 단위로 나누어 처리하기 때문에 데이터베이스나 애플리케이션의 리소스 사용 효율이 증가하며, 로직 처리 시간을 단축시킬 수 있다. MySQL 에서 페이징 쿼리는 일반적으로 LIMIT, OFFSET 구문을 사용하여 작성한다.

``` sql
select *
from subscribe
limit 500
offset 0;
```

<br>

#### 🔍 LIMIT, OFFSET

LIMIT 란 쿼리 결과의 개수를 제한하는 구문이다. 여기서 OFFSET 을 같이 활용하면 가져올 데이터의 초기 위치값을 지정할 수 있다. 위 예시에서 limit 제한으로 500개의 결과를 가져오는데 offset 이 0 이므로, 0번째 즉 처음부터 500개의 결과만 가져오게 되는 것이다. 만약 OFFSET 20; 이라면 20번째 값부터 500개의 데이터를 가져오게 된다. 쿼리로 Paging을 하고 싶다면 offset 0; 의 0 부분을 변수로 지정하여 페이지를 이동할 때마다 해당 페이지의 offset 값을 계산하여 지정해주면 된다.

<br>

### LIMIT, OFFSET 방식 쿼리의 단점

LIMIT, OFFSET 방식의 페이징 쿼리는 뒤에 있는 데이터를 읽을 수록 점점 응답 시간이 길어질 수 있다. 왜냐하면 DBMS 는 지정된 OFFSET 수만큼 모든 레코드를 읽은 이후에 데이터를 가져오기 때문이다. 아래 예시를 보자.

``` sql
SELECT *
FROM items
WHERE 조건문
ORDER BY id DESC
OFFSET 페이지번호
LIMIT 페이지사이즈
```

아래와 같은 페이징 형태가 된다.

<img src="/assets/images/pages/cs/maeil-mail/DB/스크린샷 2025-04-15 오전 11.41.03.png">

기존의 페이징 쿼리는 위와 같은 방식으로 작성되는데, 만약 OFFSET 10000; LIMIT 20; 으로 지정한다면 최종적으로 10,020 개의 행을 읽어야 한다. 마지막 20 개의 데이터를 읽기 위해 앞의 10,000 개를 행을 버리게 되는 것이다. (이 때문에 뒤로 갈수록 느려지게 된다)

### NO OFFSET

이 문제를 해결하기 위하여 OFFSET 을 사용하지 않는 페이징 쿼리를 사용한다. (NO OFFSET)

``` sql
SELECT *
FROM items
WHERE 조건문
AND id < 마지막조회ID # 직전 조회 결과의 마지막 id
ORDER BY id DESC
LIMIT 페이지사이즈
```

NO OFFSET 은 아래와 같이 페이징 번호가 없는 방식을 말한다.

<img src="/assets/images/pages/cs/maeil-mail/DB/스크린샷 2025-04-15 오전 11.42.29.png">

이전에 조회된 결과를 한 번에 건너뛸 수 있게 마지막 조회 결과의 id 를 조건문에 사용하는 것으로 이전 페이지 전체를 건너뛸 수 있다. 즉, 아무리 페이지가 뒤로 가더라도 처음 페이지를 읽은 것과 동일한 성능을 가지게 되는 것이다.

<br>

🔖 참고자료

[매일메일](https://www.maeil-mail.kr/question/245)

[LIMIT / OFFSET 쿼리](https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-LIMIT-OFFSET)

[페이징 성능 개선하기 - No Offset 사용하기](https://jojoldu.tistory.com/528)