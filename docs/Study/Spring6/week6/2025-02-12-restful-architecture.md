---
title: "RESTful 아키텍처 원칙"
parent: Week6 - RESTful 아키텍처
nav_order: -1

toc: true
toc_sticky: true

date: 2025-02-12
last_modified_at: 2025-02-12
---

# RESTful 아키텍처 원칙

## REST 란?
#### REST (REpresentational State Transfer)

### 자원의 표현 (Representational State)
자원의 상태(state)를 표현(representation)한다. 데이터와 데이터를 설명하는 메타 데이터로 구성된다.

``` http
HTTP/1.1 200 OK // 상태 코드
Content-Type: application/json // 헤더 (메타 데이터)
<Product>
  <id>p01</id>
  <name>name01</name>
  <price>100</price>
  <stock>10</stock>
</Product> // 바디 (데이터)

출처 : <그림으로 배우는 스프링 6 입문> 233p
```

### 전송 (Transfer)
자원의 표현(representational state) 을 전송한다. 즉 클라이언트와 서버 간에 representation (데이터+메타데이터) 으로 자원을 주고받는다.


## RESTful
REST 는 아키텍처 스타일의 종류 중 하나라고 보면 되고, 이러한 REST 아키텍처 스타일을 따른 시스템을 RESTful 이라고 한다.


## REST 아키텍처 스타일의 제약조건
6가지 제약조건이 있으며, 이를 만족하는 웹 서비스는 RESTful 하다.

1. Client-Server (클라이언트-서버 구조)
    - 서버와 클라이언트를 분리한다.
    - 상호 간 내부 작업을 몰라도 URL 로 자원을 식별하여 요청할 수 있다.

2. Stateless (무상태성)
    - 서버와 클라이언트가 서로의 상태를 저장할 필요가 없다. 다시 말해 서버는 API 를 요청하는 사용자의 정보를 저장하지 않는다.
    - 마찬가지로 과거의 요청 또한 저장하지 않으며, 각 요청은 독립적으로 처리된다.

3. Cacheable (캐시 처리 가능)
    - 요청에 대한 서버의 응답에 데이터가 캐시 가능한지 불가능한지 명시해야 한다.
    - 응답 헤더 cache-control 사용

4. Layered System (계층화 시스템)
    - 서버는 계층화된 시스템 아키텍처를 사용한다. (보안, 로드 밸런싱, 암호화 계층을 추가해 구조상의 유연성을 둘 수 있고, 프록시, 게이트웨이 같은 네트워크 기반의 중간 매체를 사용할 수 있게 한다)
    - 중간 계층이 요청이나 응답에 영향을 미치지 않아야 한다.

5. Code on Demand (주문형 코드, Optional)
    - 유일한 선택적 제약조건으로, 해당 조건은 지키지 못하더라도 RESTful 하다.
    - 클라이언트가 서버에 코드를 요청할 수 있다면, 요청을 받은 서버는 실행 가능한 로직 (일반적으로 자바스크립트 등 스크립트 형식의 코드) 을 제공한다.

6. Uniform Interface (인터페이스 일관성)
    - 자원에 대한 요청을 통일하고 한정적인 인터페이스로 수행하는 아키텍처 스타일이다.
    - 한정된 인터페이스란 GET, POST, PUT, DELETE 등 표준화된 HTTP 메서드를 사용하여 자원에 접근하는 것을 말한다.


## Uniform Interface
Uniform Interface 역시 아키텍처 스타일이므로 이를 만족시키기 위한 제약 조건들이 있다.

1. identification of resource (자원의 식별)
    - URI 라는 고유 식별자를 사용하여 자원을 식별한다.

2. manipulation of resources through representations (표현을 통한 자원의 조작)
    - 클라이언트가 자원을 요청할 때 자원 자체가 아닌 자원의 표현 (representation) 으로 응답한다.
    - 클라이언트가 JSON, XML 등 특정 형식으로 요청하면 서버는 요청에 맞는 표현을 제공한다.

3. self-descriptive messages (자기서술적 메시지)
    - API 문서가 REST API 응답 본문에 있어야 한다.
    - 서버나 클라이언트가 변경되더라도 메시지는 언제나 self-descriptive 하므로 API 문서를 통해 언제든 해석 가능하도록 한다.
    - 예시를 통해 알아보자.
    
    1. 응답 예시 1 : 어떤 문법과 명세로 작성했는지 표시

        ``` http
        HTTP/1.1 200 OK
        Content-Type: application/json-patch+json

        [{ "op": "remove", "path": "/a/b/c" }]

        출처 : [velog - REST API 뽀개기] 
        ``` 

    2. 응답 예시 2 : id, title 이 무엇을 의미하는지 정의한다. IANA 에 미디어 타입을 등록하고 미디어 타입 문서를 명세로 등록하면 된다. (매번 미디어 타입을 정의해야 하는 번거로움이 있다…)

        ``` http
        HTTP/1.1 200 OK
        Content-Type: application/vnd.todos+json

        [
            {"id: 1, "title": "회사 가기"},
            {"id: 2, "title": "집에 가기"}
        ]

        출처 : [velog - REST API 뽀개기] 
        ```

    3. 응답 예시 3 : id, title 이 무엇을 의미하는지 정의한 명세를 작성한다. Link 헤더에 “profile” 로 해당 명세를 링크한다. 메시지를 보는 사람은 명세를 찾아갈 수 있으므로 문서의 의미를 명확히 해석할 수 있다.

        ``` http
        GET /todos HTTP/1.1
        HOST: example.org

        HTTP/1.1 200 OK
        Content-Type: application/json
        Link: <https://example.com/docs/todos>; rel="profile"

        [
            {
                "id: 1, 
                "title": "회사 가기"
            },
            {
                "id: 2, 
                "title": "집에 가기"
            },
        ]
        출처 : [velog - REST API 뽀개기] 
        ```

4. HATEOAS (Hypermedia As The Engine Of Application State, 하이퍼링크를 통한 상태의 전이)
    - 링크(hypermedia) 에 자기 자신의 상태가 담겨야 한다.
    - 링크를 통해서 애플리케이션의 상태 전이가 가능해야 한다.
    - 상태 전이란 해당 URI에서 사용자가 다음으로 취할 수 있는 것들을 말한다.
    - 예시를 통해 알아보자. (상품 리뷰 페이지 API 응답)

    ``` http
    HTTP/1.1 200 OK
    Content-Type: application/hal+json

    {
        "data": { // HAL JSON의 리소스 필드
            "Content": "너무 감사합니다ㅎㅎ"
            "Id": "956065",
            "Name": "aaa",
            "likes: "0",
            ... 
        },
        "_links": { // HAL JSON의 링크 필드
            "self": { // 현재 api 주소
                "href": "https://www.example.com/review?page=2"
            },
            "prev": { // 이전 리뷰 페이지 api 주소
                "href": "https://www.example.com/review?page=1"
            },
            "next": { // 다음 리뷰 페이지 api 주소
                "href": "https://www.example.com/review?page=3"
            },
        },
    }

    출처 : [velog - REST API 뽀개기] 
    ```

        - 애플리케이션의 상태가 어떻게 변화할지 예측할 수 있다.
        - API 와 사용자 간의 결합이 느슨해진다.
        - 링크 정보를 동적으로 바꿀 수 있다.

<br>

RESTful 아키텍처의 원칙에 대해서 알아보았다. 이론적인 내용으로, HTTP 를 활용한 API 를 만들어 봤다면 생소한 내용은 아닐 것이다. 다음 시간에는 원칙을 바탕으로 RESTful 웹 서비스를 직접 만들어 보자.

🔖 참고자료

1. 토키 코헤이 <그림으로 배우는 스프링 6 입문>, 옮긴이 김성훈, 한빛미디어 2024.03.29

2. [(velog)REST API 뽀개기](https://velog.io/@dsa2341/REST-API-%EB%BD%80%EA%B0%9C%EA%B8%B0-6vpzm4ix#1-restrepresentational-state-transfer)