---
title: 엔티티 매핑
parent: JPA
nav_order: -3
---

# 엔티티 매핑

> JPA 에서는 클래스에 @Entity 어노테이션을 쓰면 해당 객체와 DB 테이블을 매핑해준다. (이것이 ORM 이다 !) 객체와 테이블 뿐만이 아니라 필드와 컬럼, 기본 키, 더 나아가서 연관 관계 까지 매핑하는 기능을 제공하고 있으며 예제를 통해 공부해보자.

### 🧩 엔티티 매핑 소개

| @어노테이션             | 기능               |
| ----------------------- | ------------------ |
| @Entity, @Table         | 객체와 테이블 매핑 |
| @Column                 | 필드와 컬럼 매핑   |
| @Id                     | 기본 키(pk) 매핑   |
| @ManyToOne, @JoinColumn | 연관관계 매핑      |

### 🧩 @Entity

- 해당 어노테이션이 붙은 클래스는 JPA 가 관리하며, 엔티티라 부른다. (JPA를 사용해서 테이블과 매핑할 클래스는 해당 어노테이션을 필수로 기재해야 한다.)

- 기본 생성자가 필수이다.

- final 클래스, enum / interface / inner 클래스에 사용할 수 없다.

- 필드에 final 사용할 수 없다.

| 속성 | 기능                                                                                |
| ---- | ----------------------------------------------------------------------------------- |
| name | JPA 에서 사용할 엔티티 이름을 지정한다. 기본적으로는 클래스 이름을 그대로 사용한다. |

```java
@Entity(name = "MBR")
public class Member {
    //...
}
```

### 🧩 @Table

- 엔티티와 매핑할 테이블을 지정한다.

| 속성              | 기능                                                                              | 기본값                  |
| ----------------- | --------------------------------------------------------------------------------- | ----------------------- |
| name              | 매핑할 테이블 이름                                                                | 엔티티 이름을 사용한다. |
| catalog           | 데이터베이스의 catalog 매핑                                                       |
| schema            | 데이터베이스의 schema 매핑                                                        |
| uniqueConstraints | DDL 생성 시 유니크 제약 조건을 지정할 수 있다. (제약조건의 이름을 지정할 수 있음) |

### 🧩 필드와 칼럼 매핑

- 필드와 칼럼을 매핑할 때 쓰는 어노테이션들 !

| 어노테이션  | 설명                                         |
| ----------- | -------------------------------------------- |
| @Column     | 컬럼 매핑                                    |
| @Temporal   | 날짜 타입 매핑                               |
| @Enumerated | enum 타입 매핑                               |
| @Lob        | BLOB, CLOB 매핑                              |
| @Transient  | 특정 필드를 칼럼에 매핑하지 않음 (매핑 무시) |

- 각 어노테이션은 다양한 속성을 가진다.

🍥 @Column

| 속성                  | 설명                                                                                                                                                         | 기본값                    |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------- |
| name                  | 필드와 매핑할 테이블의 칼럼 이름                                                                                                                             | 객체의 필드 이름          |
| insertable, updatable | 등록, 변경 가능 여부 (영속성 컨테이너 안에서 변경이 일어날 경우 디비에 반영할지 여부를 체크한다. true 면 반영이 된다는 뜻 !)                                 | TRUE                      |
| nullable(DDL)         | null 값의 허용 여부를 설정한다. false 로 설정하면 DDL생성 시 not null 제약 조건이 붙는다.                                                                    | TRUE                      |
| unique(DDL)           | 칼럼 하나에 간단히 유니크 제약조건을 걸 때 사용한다. 다만, 이 속성은 제약조건 이름이 무작위로 설정되어 개발자가 에러 코드에서 확인하기 어렵다는 단점이 있다. |
| columnDefinition(DDL) | 데이터베이스 칼럼 정보를 직접 줄 수 있다. (ex. varchar(100) default 'EMPTY')                                                                                 |
| length(DDL)           | 문자 길이 제약조건, String 타입에만 사용한다.                                                                                                                | 255                       |
| precision, scale(DDL) | BigDecimal 타입에서 사용한다. 아주 큰 숫자나 정밀한 소수를 다뤄야 할 때만 사용한다.                                                                          | percision = 19, scale = 2 |

🍥 @Enumerated

| 속성  | 설명                                                                                                                                                            | 기본값           |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| value | EnumType.ORDINAL : enum 순서를 데이터베이스에 저장 (식별하기 어려움...) / EnumType.STRING : enum 이름을 데이터베이스에 저장 (이 옵션을 사용하는 것이 권장된다.) | EnumType.ORDINAL |

### 🍥 사용 예시를 봅시다 !

```java
    @Id
    private Long id;

    @Column(name = "name")
    private String username;

    private Integer age;

    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    private LocalDate createDate2;

    @Lob
    private String description;

    @Transient //DB와 관계없이 그냥 메모리에서만 쓰고 싶은 필드..
    private int temp;
```

### 🧩 기본 키 매핑

- 기본 키를 매핑하는 어노테이션은 크게 두 가지가 있다.

- 권장하는 식별자 전략은 Long형과 대체키를 사용하는 전략이다.

| 어노테이션      | 설명                                                                                         |
| --------------- | -------------------------------------------------------------------------------------------- |
| @Id             | 필드 위에 붙이는 것으로 직접 기본 키로 지정한다.                                             |
| @GeneratedValue | 기본 키를 자동 생성해주는 역할을 하며, strategy 속성으로 기본 키 생성 전략을 지정할 수 있다. |

🍥 @GeneratedValue

| 속성     | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | 사용 DB          |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------- |
| IDENTITY | 기본 키를 필드와 매칭시키지 않고 데이터베이스가 자동으로 생성하도록 위임한다. 데이터를 넣을 때 기본 키 값이 null 로 들어가며 insert 되고 나서야 그 행의 기본 키가 무엇인지 확인할 수 있다. 여기서 문제는... JPA에서 영속성 컨텍스트의 1차 캐시에 저장되는 엔티티는 (key : 기본 키, value : 객체) 로 저장이 되는데 쿼리문이 실행되는 시기가 커밋되는 시점이라는 것이다. 즉 커밋되지 않으면 그 엔티티를 1차 캐시로 저장할 수가 없다. 그래서 IDENTITY 전략이 선택된 경우에만, em.persist 되는 시점에 insert 쿼리가 실행되고, JPA 는 자체적으로 디비에서 생성된 기본 키 값을 가져와서 1차 캐시에 저장한다. | MYSQL            |
| SEQUENCE | 데이터베이스 시퀀스 오브젝트를 사용해서 개발자가 지정한 기본 키 전략대로 기본 키를 생성한다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | ORACLE           |
| TABLE    | 키 생성용 테이블을 만들어서 사용한다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | 모든 DB에서 사용 |

```java
// IDENTITY 전략
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

```java
// SEQUENCE 전략
@Entity
@SequenceGenerator(
    name = “MEMBER_SEQ_GENERATOR",
    sequenceName=“MEMBER_SEQ",//매핑할 데이터베이스 시퀀스 이름
    initialValue = 1, allocationSize = 50)
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
        generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
}
```

- SEQUENCE 전략은 시퀀스를 한 번 호출할 때마다 기본 키가 증가한다. (예제에서는 1부터 증가하도록 만들었다.)
- 문제는 매번 insert 수행할 때마다 시퀀스를 호출하기 때문에 성능 저하의 원인이 된다는 것이다.
- 그래서 allocationSize 속성을 이용하여 시퀀스 호출 한 번에 가져올 값을 세팅할 수 있다.
- 예제에서처럼 작성하면 처음 insert 를 수행할 때 시퀀스가 호출되어 50까지의 수를 가져오고, 그 이후 insert 될 때는 메모리에 저장되어 있는 수를 가져다가 기본 키로 사용한다.
- 50까지 다 사용하게 되면 다시 한 번 시퀀스를 호출해서 100까지의 수를 가져와서 메모리에 저장한다.
- 이렇게 하면 네트워크를 통한 디비와의 통신 횟수가 줄어들기 때문에 성능 최적화에 사용된다.

```sql
-- TABLE 전략 (기본 키 생성용 테이블 만들기)
create table MY_SEQUENCES (
sequence_name varchar(255) not null,
next_val bigint,
primary key ( sequence_name )
)
```

```java
@Entity
@TableGenerator(
name = "MEMBER_SEQ_GENERATOR",
table = "MY_SEQUENCES",
pkColumnValue = “MEMBER_SEQ", allocationSize = 1)
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
    generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
}
```

### 다음으로 ...

> 이런 식으로 키 값을 매핑할 수 있지만, 여기에만 그친다면 데이터베이스를 기반으로 객체를 설계하는 방식이 되어버린다. 즉 객체지향적이지 못한 설계가 되어버린다. 다음 시간에는 연관 관계를 매핑하는 방법을 공부해보자.
