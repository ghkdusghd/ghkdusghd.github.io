---
title: "[NCBT] JUnit 을 활용한 테스트 코드 작성 (1)"
parent: NCBT
nav_order: -12

tags:
  - [junit]

toc: true
toc_sticky: true

date: 2025-01-20
last_modified_at: 2025-01-20
---

### 들어가며

약 2개월간의 NCBT 프로젝트 개발이 끝나고 배포를 하는 과정에서 JDBC 연결 문제가 발생했다. 물론 문제는 잘 해결했지만... 이러한 연결 문제는 개발 초보자들이 흔히 겪는 이슈인 만큼 더욱더 테스트 코드가 있었더라면 빌드 시 문제가 되는 지점을 빠르게 파악하고 해결할 수 있지 않았을까 하는 생각이 들었다. 이번 문제 뿐만 아니라 다른 기능들에 대해서도 결함을 조기에 발견할 수 있도록, 그리고 추후 CI/CD 환경에서 같은 문제가 발생하지 않도록 테스트 코드를 작성해보자.

이 글에서는 테스트 코드를 작성하는 과정과 세부 코드를 정리해 보고자 한다.

<br>

### 👩🏻‍💻 테스트 목표

1. 애플리케이션의 기능이 올바르게 동작하는지 확인한다.

    - 실패 케이스에 대한 에러 핸들링 : 성공 케이스보다 실패 케이스에 집중해야 한다. 예상 실패 시나리오를 세우고 실패했을 때 어떻게 동작할지 만들어두자. 자연스럽게 기존 코드들에 대한 리팩토링이 될 것이다.

2. 테스트 비용을 최대한 줄인다.

    - 단위 테스트와 통합 테스트 수행 : TDD 개발의 경우 대개 수시로 빠르게 검증이 가능한 단위 테스트를 선호하지만, 나의 경우 개발이 끝난 서비스 직전 단계에서 기능이 올바르게 동작하는지 검증하고자 하는 목적도 있기에, 목 데이터를 사용하지 않는 통합 테스트를 적절히 섞어서 수행할 것이다.

    - 통합 테스트의 단점 : 통합 테스트로 모든 기능이 실제 환경에서 잘 작동하는지 테스트하면 좋겠지만, 여러 모듈을 포함해서 테스트하기 때문에 속도가 느리고 무거워진다. 그리고 단위 테스트보다 디버깅하기 더 어렵다는 단점도 있다. 때문에 테스트를 수행함에 앞서 실제 환경에서의 테스트가 필요한 메소드가 무엇인지 신중하게 고민하고 진행하자.

3. 데이터 일관성을 지킨다.

    - 통합 테스트를 진행할 때, 테스트 코드로 인해 데이터가 변경되면 안 된다. @Transactional 어노테이션을 사용해서 테스트 메소드가 실행될 때 마다 무조건 롤백시키자.

4. 사전에 테스트 시나리오를 상세히 작성한다.

    테스트 시나리오를 상세히 작성하는 것으로 실패 사례에 대해서 깊이 고민해 볼 수 있고, 그 과정에서 예상치 못한 실패 사례를 발견할 수 있다.

    <details>
    <summary><b style="color: green;">  테스트 시나리오 접기/펼치기 (UT : 단위테스트, AIT : 통합테스트)</b></summary>
    <div markdown="1">

    <img src="/assets/images/pages/projects/ncbt/스크린샷 2025-02-10 오전 9.53.19.png">

    </div>
    </details>

5. 결함을 관리한다.

    - 테스트 단계에서 발견한 결함은 문서화하여 관리한다. 
    
    - 적절한 조치를 취하고 테스트를 다시 수행한다.

<br>

### 💡 사전지식

<br>

> #### 1단계 : 테스트 케이스 명명 규칙

테스트 케이스 명명 규칙을 찾아보니 크게 세 가지가 있는 듯 하다.
- 메소드 이름을 기준으로 테스트 케이스를 작성하는 방법
- 테스트하고자 하는 기능을 기술하듯이 적는 방법
- given when then 등의 단어를 활용하는 방법

이 중에서 나는 2번 규칙을 선택했다. 메소드 이름을 기술하듯이 적으면 네이밍이 어렵고 사람마다 헷갈릴 수 있다는 단점이 있긴 하지만... 테스트 시나리오를 문서화할 것이기 때문에 테스트 케이스에 해당하는 테스트 ID 를 적어두면 팀원과의 의사소통 과정에서 무리가 없을 것 같아 선택했다.

<br>

> #### 2단계 : 테스트 관련 어노테이션 정리

1. 테스트 클래스에 붙이는 어노테이션

    | 어노테이션 | 주요 역할 | options |
    |---|---|---|
    | @SpringBootTest | @SpringBootApplication 을 찾아서 테스트를 위한 Bean 을 생성하고, @MockBean 으로 정의된 Bean 을 찾아서 대체시킨다 | webEnvironment 옵션으로 테스트 환경을 선택할 수 있다. (실제 WAS 를 띄우려면 DEFINE_PORT, MOCK 서버를 띄우려면 default 설정) |
    | @WebMvcTest | 컨트롤러 동작만 확인하는 경우 사용한다. 컨트롤러 관련한 어노테이션 및 클래스만 로딩한다. (잘 사용되진 않는다고 한다...) | |
    | @AutoConfigurationMockMvc | @SpringBootTest 와 같이 사용되며, Service 와 Repository 등의 Bean 도 같이 로딩한다. | |
    | @DisplayName | 가시성을 위해 테스트명을 직접 입력해줄 수 있다. | "" 따옴표로 내용을 적으면 된다. |

2. 테스트 필드에 붙이는 어노테이션

    | 어노테이션 | 주요 역할 |
    |---|---|
    | @Test | JUnit 은 @Test 어노테이션이 붙은 메소드를 자동으로 실행하고 결과를 출력한다. |
    | @Mock vs @MockBean | 의존성 관리 주체가 다르다. MockBean 으로 정의한 객체는 스프링 전용 Mock 이라고 보면 되는데, AppCtx 구성시 BeanFactory 에 실제 Bean 대신 MockBean 을 생성하여 로딩하고, 그냥 Mock 으로 정의한 객체는 BeanFactory 에 포함되지 않고 코드 상에서 @InjectMock 이나 생성자, setter 로 의존성을 주입한다. |
    | @Spy vs @SpyBean | Mock 은 가상 객체여서 실제 소스코드가 실행되지 않고 결과에 대한 리턴만 stub```*1```할 수 있다. 일부 메소드는 stub 하고 다른 메소드는 실제 소스가 수행되는 테스트를 구성하고 싶을 때 쓰는 것이 @Spy 다. Mock 을 원하는 대상 메소드만 stub 처리하면 다른 메소드는 실제 소스코드가 수행된다. |

    ```*1 stub : Mock 객체에 내가 테스트하고자 하는 상황을 주고 그 때의 리턴값을 정해두는 것. 예를 들어 sendMail 이라는 메소드에 특정한 값(보내는 대상, 메시지 등)이 들어가면 true 를 반환하도록 정해두는 것을 Stubbing 한다고 한다.``` 


<br>

### 📎 테스트 케이스 작성 및 실행

### 통합 테스트

실제 DB 와 연결이 되는지 확인하기 위해 Mock 을 사용하지 않는 통합테스트를 수행한다. 주목할 점은, 스프링 부트의 경우 application.yml 에 있는 환경 변수를 애플리케이션 실행과 동시에 로드하기 때문에 AIT_01_JDBC 연결 테스트는 자동적으로 수행된다 할 수 있다. 그러므로 코드로 작성하지 않고 넘어갔다.

``` java
@SpringBootTest
public class MyBatisTest {

    @Autowired
    private DataSource dataSource;

    @Autowired
    private UserMapper userMapper;

    @Test
    @DisplayName("AIT_02_HikariCP 구축 테스트")
    public void ConnectionPoolTest() {
        Assertions.assertNotNull(dataSource);
        try {
            Connection connection = dataSource.getConnection();
            Assertions.assertEquals(false, connection.isClosed());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Test
    @Transactional
    @DisplayName("AIT_03_MyBatis 구축 테스트")
    public void MyBatisTest() {
        // given
        LoginDTO testUser = new LoginDTO();
        testUser.setUsername("tester");
        testUser.setPassword("password");
        testUser.setEmail("tester@tester.com");
        testUser.setRoles("USER");

        // when
        userMapper.insertUser(testUser);

        // then
        List<User> allUsers = userMapper.findAll();
        User user = allUsers.get(allUsers.size() - 1);
        Assertions.assertEquals("tester", user.getUsername());
    }

}
```

<img src="/assets/images/pages/projects/ncbt/스크린샷 2025-02-04 오후 2.11.34.png">

결과는 전부 성공.

<br>

### 단위 테스트와 리팩토링

테스트 시나리오를 작성하면서 개발할 때 미처 발견하지 못했던 에러 사항이 있었기에 먼저 코드를 리팩토링한 후 테스트를 진행했다.

> #### 리팩토링

1. 예외 처리 추가

    - 코드의 가독성과 재사용성을 위해 최상위 계층에서 예외를 잡아 처리하게 했다.
    - 오류를 더 쉽게 추적하기 위해 Layer별 역할에 맞게 예외를 던지도록 했다. 

    | 계층 | 로직 | 예외 |
    |---|---|---|
    | 프레젠테이션 계층 | 깃허브 로그인 | 네이버와 다르게 RestTemplate 의 exchange() 메서드를 사용했는데, 이 경우 통신 관련 에러가 발생하면 HttpClientErrorException 으로 처리되기 때문에 컨트롤러에서 예외를 잡도록 했다. |
    | 비즈니스 로직 계층 | 네이버 로그인 | 인가 코드를 받아 공급자에게 접근 코드를 요청하는 로직에서 에러가 발생할 수 있으니 받아온 접근 코드가 null 이면 CustomException 을 던지도록 수정했다. |
    | 데이터 액세스 계층 | Refresh Token 저장 | 추후 Refresh Token 검증을 위해 반드시 DB 에 저장해야 하는데, 혹시 저장이 되지 않았다면 보안 문제가 발생하므로 DataAccessException 예외를 던지도록 추가했다. |

2. 하나의 메서드는 하나의 기능만 수행하도록 분리

    - 기존에 서비스 계층에 만들어 둔 소셜로그인 메서드 하나에서 공급자와 통신을 하고 DB 에 조회도 하는 등 많은 로직이 들어가 있는 점이 복잡하게 느껴졌다. DB 조회하는 로직을 따로 분리하여 다른 회원가입 로직에서도 사용할 수 있도록 재사용성을 높였다.
    - 기존 메서드 : RestTemplate 객체로 공급자로부터 사용자 정보를 가져온다.
    - 분리한 메서드 : 사용자 정보가 DB 에 존재하는지 확인하고, 존재하지 않으면 Insert 한다.
    - 컨트롤러에서 기존 메서드를 호출하면 서비스 계층에서 사용자 정보를 받아와 OauthUserDTO 를 생성하여 리턴한다. 그 후 분리한 메서드에 DTO 를 전달하여 DB 를 조회하고, insert 한다.

<br>

### 느낀점

개발할 때는 에러 핸들링이고 뭐고 기능부터 빨리빨리 구현해야 한다는 알 수 없는 압박감(?) 에 쫓겨서 숲을 보지 못했던 것 같다. 처음에는 테스트 코드를 통한 결함 줄이기가 목표였지만 좋은 에러 핸들링이란 무엇인가 공부가 되었고, 직접 리팩토링하면서 개선하는 과정에서 재미와 뿌듯함을 느꼈다. 다음 글에서는 테스트 케이스를 자세하게 정리하고 결함 관리를 어떻게 했는지 작성해보자.

<br>

🔖 참고자료

[효율적인 JUnit 사용 방법과 유용한 팁](https://yozm.wishket.com/magazine/detail/1748/)

[SpringBootTest 정리](https://velog.io/@leejisoo/SpringBootTest-%EC%A0%95%EB%A6%AC)

[JDBC 단위테스트](https://steady-record.tistory.com/entry/Spring-JUnit%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%8B%A8%EC%9C%84%ED%85%8C%EC%8A%A4%ED%8A%B8JDBC-HikariCP-MyBatis)

[좋은 예외 처리란?](https://jojoldu.tistory.com/734)