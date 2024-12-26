---
title: 연관관계 매핑
parent: JPA
nav_order: -4
---

# 연관관계 매핑

> 객체와 테이블 연관관계의 차이를 생각해보자. 자바에서는 연관관계를 갖는 객체의 경우 '참조'하는 방식으로 값을 가져오지만 DB 테이블은 외래 키로 제약 조건을 설정하여 참조 무결성을 지킨다. 다시 말해 연관관계를 갖는 객체(테이블)의 값을 가져오는 방식이 다른 것이다.

### 🧩 연관관계가 필요한 이유

- 앞서 살펴본 방법대로, 객체를 테이블에 맞춰 모델링한다고 생각해보자.

- 테이블은 외래 키 설정으로 연관관계를 맺었지만 이를 그대로 자바 객체로 옮기면 객체 간의 연관관계를 설정하기 어려워진다.

<img src="/assets/images/pages/cs/jpa/02. Member, Team table.png">
<span style="color: #808080">출처: 자바 ORM 표준 JPA 프로그래밍 - 기본편 (인프런)</span>

&nbsp;

🍥 엔티티 매핑

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @Column(name = "TEAM_ID")
    private Long teamId;
}

@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;
    private String name;
}
```

&nbsp;

🍥 CRUD

- 이렇게 매핑한 이후 CRUD 작업에 들어갔을 때 아래와 같은 방식으로 작업을 하게 되는데,

- 연관관계에 있는 객체를 직접 참조하지 못하고, id 라는 식별자로 조회해야 한다. (객체지향적인 방법이 아니다...)

```java
// TeamA 생성
Team team = new Team();
team.setName("TeamA");
em.persist(team);

// TeamA에 멤버 저장
Member member = new Member();
member.setName("memberA");
member.setTeamId(team.getId());
em.persist(member);

// 저장한 팀과 그 팀의 멤버를 조회
Member findMember = em.find(Member.class, member.getId());
Team findTeam = em.find(Team.class, team.getId());
```

- 결론적으로, 객체를 테이블에 맞추어 데이터 중심으로 모델링하면 협력 관계를 만들 수 없다.

### 🧩 단방향 연관관계

> 연관관계 매핑에는 '단방향'과 '양방향' 방식이 있다. 우선 '단방향' 이란 연관관계에 있는 두 개의 객체 중 하나에만 참조값을 넣어주는 것을 말한다. 예시를 보자면 하나의 Team 에 여러 명의 Member 가 연관관계에 있으므로, Member 객체의 필드에 Team 객체를 넣어주면 단방향 매핑이 완성된다.

- Team, Member 테이블을 객체 지향적으로 다시 설계해보자.

- @ManyToOne ? Member (본인) 와 Team (상대) 의 관계를 알린다. (다대일 관계)

- @JoinColumn ? 실제 MEMBER 테이블의 어떤 칼럼과 매핑할 것인지 명시해준다.

```java
public class Member {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

&nbsp;

🍥 CRUD

```java
// 저장
Team team = new Team();
team.setName("teamA");
em.persist(team);

Member member = new Member();
member.setName("memberA");
member.setTeam(team);
em.persist(member);
```

### 🧩 양방향 연관관계

> 위 예제는 Member 객체에서 해당 객체가 어떤 team 에 소속되어 있는지 알 수 있지만, 반대의 경우는 알 수가 없다. 그래서 Team 객체의 필드에도 Member 객체를 참조값으로 넣을 수 있다. 이렇게 되면 객체가 서로를 참조하고 있으므로 양방향 연관관계가 된다.

- @OneToMany ? 마찬가지로 객체와의 관계를 정의한다. (Member와는 다대일 관계)

- mappedBy ? 연관관계의 주인과 관련이 있는 속성.

- NullPointException 을 방지하기 위해 컬렉션을 초기화시켜준다.

```java
public class Team {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = ArrayList<>();
}
```

&nbsp;

🍥 CRUD

```java
// 양방향 매핑한 경우, 연관관계의 주인이 아닐 경우 [조회] 만 가능하다.
Team findTeam = em.find(Team.class, team.getId());
int memberSize = findTeam.getMembers().size();
```

- mappedBy 는 연관관계의 주인이 아닌 경우에만 사용하는 속성이다.

- 해당 속성을 가진 객체로 생성, 삭제를 해도 JPA 에서는 그 코드를 읽지 않는다.

- 오로지 조회만 가능하다.

### 🧩 연관관계의 주인

> 테이블의 경우 외래 키 하나로 두 테이블의 연관관계를 관리하지만, 객체는 사실 서로 다른 단방향 관계 2개로 연관관계를 관리하고 있다. 그렇기 때문에 '연관관계의 주인' 을 정해서 실제 DB 테이블과 매핑할 객체가 무엇인지를 하나 정해야 한다. 그렇다면 어떤 객체를 주인으로 정해야 할지 고민이 될 것이다. 이 때 간편한 방법은 외래 키가 있는 테이블의 엔티티를 주인으로 정하는 것이 좋다. 예제에서는 Member.team 이 연관관계의 주인이 되는 것이다.

#### 🍥 주의 ! 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정해야 한다.

- DB 에서 "TeamA" 를 조회해서 거기에 소속된 Members 를 검색해보자.

```java
// 저장
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setTeam(team);
em.persist(member);

// 영속성 컨텍스트를 비워서 1차 캐시 삭제
em.flush();
em.close();

// 조회 (1차 캐시가 비어있으므로 DB에서 조회해서 가져온다)
Team findTeam = em.find(Team.class, team.getId());
List<Member> findMembers = findTeam.getMembers();
```

- 이 경우 문제 없이 team 에 소속된 member 객체가 출력될 것이다.

- 그런데 이상하다. team 객체에서 member 객체를 추가하지 않았는데 어떻게 출력이 될까? => 연관관계의 주인인 Member.team 에서 setTeam 으로 team 을 insert 해두었기 때문에 테이블에 저장이 되었고, 영속성 컨텍스트의 1차 캐시를 지워버리면 JPA 는 DB 에서 데이터를 조회하므로 문제 없이 가져올 수 있는 것이다.

- 만약, 1차 캐시를 비우지 않았다면 ? => 1차 캐시의 Team 객체에는 Member 객체가 없으므로 데이터를 가져올 수 없다.

- 이러한 문제가 생길 수 있어서 항상 양쪽에 값을 설정해주는 것이 좋고, 이를 아래와 같이 로직으로 만들어두면 실수를 줄일 수 있다.

```java
// 1. 역방향(주인이 아닌 방향)에 값을 직접 입력 하거나,
team.getMembers().add(member);

// 2. 아예 로직으로 구현하여 한 번에 양쪽에 입력되게 하는 방법이 있다. (2번 방법을 더 추천)
public class Member {
    ...
    public changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}
```

### 🧩 실전에서는 ...

- 테이블 연관관계를 토대로 객체 연관관계를 설계하게 되므로, 테이블부터 잘 설계하는 것이 중요하다는 생각이 든다.

- 왠만하면 단방향으로만 설계하는 것이 좋다.

- 일단 단방향으로 설계한 이후, 필요하면 객체를 생성해서 조회해도 된다.
