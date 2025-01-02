---
title: DB Replication에 대해서 설명해주세요.
parent: 데이터베이스
nav_order: -1
---

<div style="text-align: center; display: flex;
    flex-direction: column;
    align-items: center;">
    <h5>오늘의 질문</h5>
    <div style="color: black; background-color: #F0F3F5; border-radius: 5px; width: 50%; padding: 20px;">
    DB Replication에 대해서 설명해주세요.
    </div>
</div>

---

### 두 개 이상의 DBMS 를 사용하여 Master / Slave 구조를 활용하여 DB 부하를 분산시키는 기술.

보통 Master DB 에는 Insert, Update, Delete 작업을 수행하도록 하고 Slave DB 에는 Select 작업을 수행하도록 한다. 이 때, Slave DB 의 데이터는 Master DB 의 데이터를 복제하여 만들어진다. Select 작업을 따로 빼서 구성하는 이유는, '조회' 작업이 보통 시간이 많이 걸리기 때문이다. 이 시간동안 다른 작업을 하지 못하게 되니 병목 현상의 주요 원인이 되고, 대용량 트래픽이 발생하는 서비스에서 분산 처리를 위해 Replication 기술을 활용한다.

<br>

### 바이너리 로그 기반 복제 (Binary Log)

Slave 에 데이터를 복제하는 방법 ! MySQL 에서 제공하는 바이너리 로그를 기반으로 복제한다. 바이너리 로그란, MySQL 서버에서 Create ,Alter, Drop 과 같은 작업을 수행하면 MySQL 은 그 변화된 이벤트를 기록하는데, 이러한 변경사항을 담고 있는 이진 파일을 바이너리 로그라 한다. (show or select 같은 조회 문법은 제외된다)

<br>

### Replication 동작 방식

MySQL 의 Replication 은 기본적으로 비동기 방식으로 동작한다. Master 에서 변경되는 데이터에 대한 이력이 로그에 기록되면 Replication Master Thread 가 비동기적으로 이를 읽어서 Slave 에 전송한다.


> #### 순서로 이해해보자 !

``` markdown
1. Commit 발생

2. Connection Thread 에서 스토리지 엔진에게 해당 트랜잭션에 대한 Prepare (Commit 준비) 를 수행

3. Commit 을 수행하기 전에 Binary Log 에 변경사항 기록

4. 스토리지 엔진이 트랜잭션 Commit 수행

5. Master Thread 는 비동기적으로 Binary Log 를 읽어서 Slave 에 전송

6. Slave 의 I/O Thread 는 Master 로부터 수신한 변경 데이터를 Relay Log 에 기록

7. Slave 의 SQL Thread 는 Relay Log 에 기록된 변경 데이터를 읽어서 Slave 의 스토리지 엔진에 적용
```

<br>

### Replication 의 장점

- Select 성능 향상

    - 대개 조회 작업에서 시간을 많이 잡아먹게 되는데, Replication 을 구성하면 N 개의 Slave 를 가질 수 있기 때문에 조회 작업에 대한 부하가 분산된다.

- 데이터 백업

    - Master 의 내용을 그대로 복제하기 때문에, 원본 데이터가 날아가더라도 N 개의 Slave 중 하나를 Master 로 사용하면 되기에 백업 용도로 사용할 수 있다.

<br>

### Replication 의 단점

- 데이터 정합성을 보장할 수 없음

    - Slave 는 Master 데이터를 복사하여 사용하기 때문에 그것이 정말 원본과 동일한, 완벽한 데이터인지 보장할 수 없다. 예를 들어 Slave 가 Master 의 쿼리 처리량을 따라가지 못한다면... 데이터 정합성이 보장되지 않을 수 있다.

- Binary Log 관리

    - Master 는 Binary Log 가 무분별하게 쌓이는 것을 막기 위해 보관 주기를 설정할 수 있지만 Slave 까지 관리하지 못한다. 때문에 Master 에서 Binary Log 를 삭제한다 하더라도 Slave 의 Binary Log 까지 삭제하지는 못한다.

<br>

### 정리

DB 는 기본적으로 Select 작업에서 자원 소모가 많기 때문에 Select 작업이 많아 성능 이슈가 발생하는 상황이라면 가장 먼저 고려해볼 만한 전략이다. 현재 수많은 기업에서 도입하고 있으며 분산 처리의 핵심 기술로 자리잡았다. 다만, Replication 은 단점 또한 치명적이기에 만능이라 할 수는 없다는 점에 유의하자 !

<br>

🔖 참고자료

[매일메일](https://www.maeil-mail.kr/question/109)

[DB Replication 개념 정리](https://velog.io/@zpswl45/DB-Replication-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC#replication%EC%9D%98-%EB%8B%A8%EC%A0%90)