---
title: 고급 매핑
parent: JPA
nav_order: -5
---

# 고급 매핑 (상속관계 매핑)

> 관계형 데이터베이스에는 상속 관계가 없다. 다만 논리적으로 슈퍼타입 서브타입 관계라는 모델링 기법이 객체의 상속과 유사하다. 상속관계 매핑이란, 객체의 상속구조와 DB 의 슈퍼타입 서브타입 관계를 매핑하는 것으로, 3가지 전략으로 매핑할 수 있다.

### 🍥 주요 어노테이션

| 어노테이션                         | 설명                                                                                                                                                                                 | 전략                                                                                                         |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| @Inheritance                       | 상속관계를 매핑할 때 부모 클래스에 달아줘야 한다.                                                                                                                                    | JOINED: 조인 전략, SINGLE_TABLE : 단일 테이블 전략, TABLE_PER_CLASS : 구현 클래스마다 테이블을 생성하는 전략 |
| @DiscriminatorColumn(name="DTYPE") | 부모 클래스에 달아준다. DB에서는 상속관계가 없기 때문에 insert된 데이터가 어떤 서브타입의 데이터인지 확인하기가 어렵다. 그래서 DTYPE 을 따로 설정하여 운영에서 확인하기 편하게 한다. | name 은 아무거나 넣어도 상관없지만 default 가 DTYPE 이기도 하고 보통 DTYPE 을 쓴다.                          |
| @DiscriminatorValue("name")        | 자식 클래스에 달아준다. 슈퍼타입에 insert 될 DTYPE 이름을 설정할 수 있다. 필수는 아님.                                                                                               | default 로는 클래스명이 들어간다.                                                                            |

### 🧩 　조인 전략

<img src="/assets/images/pages/cs/jpa/03. Join Strategy.png">
<span style="color: #808080">출처: 자바 ORM 표준 JPA 프로그래밍 - 기본편 (인프런)</span>

&nbsp;

- 슈퍼타입의 PK 를 서브타입이 FK 로 참조하는 구조로 설계한 경우 조인 전략을 사용할 수 있다.

- 외래 키 참조 무결성 제약조건을 활용할 수 있다는 장점이 있다.

```java
// 슈퍼 타입 (부모 클래스) : 부모 클래스를 사용할 일이 없을 경우 추상 클래스로 선언한다.

@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
public abstract class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;

}

// 서브 타입 (자식 클래스)

@Entity
public class Album extends Item {
    private String artist;
}

@Entity
public class Movie extends Item {
    private String director;
    private String actor;
}

@Entity
public class Book extends Item {
    private String author;
    private String isbn;
}
```

&nbsp;

### 🧩 　단일 테이블 전략

<img src="/assets/images/pages/cs/jpa/04. Single Table Strategy.png">
<span style="color: #808080">출처: 자바 ORM 표준 JPA 프로그래밍 - 기본편 (인프런)</span>

&nbsp;

- 모든 서브 타입의 칼럼을 부모 타입에 합쳐서 단일 테이블로 설계한 경우에는 단일 테이블 전략을 사용한다.

- 이 경우에는 조인이 필요 없으므로 일반적으로 조회 성능이 빠르고, 쿼리가 단순하다.

- 다만, 자식 클래스가 매핑하는 컬럼은 모두 null 을 허용해야 한다는 단점이 있다.

- 단일 테이블 전략의 경우, 식별을 위한 DTYPE 이 반드시 필요해서 개발자가 설정하지 않아도 하이버네이트가 자동으로 생성한다.

```java

// 슈퍼 타입 (부모 클래스)

@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;

}

// 서브 타입 (자식 클래스)

@Entity
public class Album extends Item {
    private String artist;
}

@Entity
public class Movie extends Item {
    private String director;
    private String actor;
}

@Entity
public class Book extends Item {
    private String author;
    private String isbn;
}
```

&nbsp;

### 🧩 　구현 클래스마다 테이블 생성 전략

<img src="/assets/images/pages/cs/jpa/05. Table per class Strategy.png">
<span style="color: #808080">출처: 자바 ORM 표준 JPA 프로그래밍 - 기본편 (인프런)</span>

&nbsp;

- 서브 타입에 슈퍼 타입의 칼럼을 전부 넣어서 각각 별개의 테이블을 생성하는 경우 사용할 수 있는 전략이다.

- 그러나 일반적으로 권장되지 않는다 ...

&nbsp;

### 🍥 　@MappedSuperclass

- 상속 관계에서 사용하는 매핑은 아님 ❌

- 모든 클래스에서 공통으로 사용하는 매핑 정보가 있을 때 중복 입력이 힘들기 때문에 따로 클래스를 만들어 상속받는 형식으로 사용하게 되는데, 그럴 때 붙이는 어노테이션이다.

- 공통 매핑 정보를 담은 클래스는 직접 생성해서 사용할 일이 없으므로 abstract 으로 만드는 것을 권장한다.

- 마찬가지로 그저 공통 정보만 들어있는 클래스이기 때문에 엔티티가 아니고, 테이블과 매핑하는게 아니다.

```java

// 공통 정보 클래스

@MappedSuperclass
public abstract class BaseEntity {

    private String createdBy;
    private LocalDateTime createdDate;

}

// 공통 정보를 사용하는 클래스들은 전부 상속받아서 사용

@Entity
@Inheritance
public class Item extends BaseEntity {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;

}
```
