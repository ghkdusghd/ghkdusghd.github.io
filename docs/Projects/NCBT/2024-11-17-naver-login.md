---
title: "[OAuth2] react + spring boot 로 네이버 소셜 로그인 구현"
parent: NCBT
nav_order: 3

tags:
  - [oauth2, naver]

toc: true
toc_sticky: true

date: 2024-11-17
last_modified_at: 2024-11-17
---

> 드디어 네이버 로그인 구현에 성공했다. 소셜 로그인 구현 방식을 잘 몰라서 공식 문서도 참고하고 블로그 글도 참고했는데 하나부터 열까지 자세하게 설명해둔 글이 없어서 구현하는데 삽질을 많이 했다. 거의 1주일만에 구현에 성공해서 기쁜 마음에 정리해본다.

### 1️⃣ Oauth2 로그인 과정 (네이버)

나는 네이버로 구현했기에 네이버 로그인 방식을 기반으로 정리해보려고 한다.

🔥 내가 초기에 이해한 것 (실패)

- Spring Security 의 application.yml 파일에 네이버 로그인 관련 설정 정보들을 넣어두고 Security Filter Chain 에 Oauth2Login 체인을 넣으면 스프링이 자동으로 네이버 인증 과정을 진행해준다.

- 네이버에서 받아온 유저 정보를 스프링이 OAuth2UserRequest 객체로 만들어 주고, DefaultOAuth2UserService 를 상속받는 Oauth2 관련 서비스 로직을 만들어서 해당 객체로 DB 조회 등 수행하면 된다.

- 따라서 리액트에서는 네이버 로그인을 제공해주는 서비스url 인 localhost:8080 으로 사용자를 전송해주기만 하면 된다. (서비스 url 은 네이버 로그인 API를 신청할 때 설정할 수 있다)

- 위 과정을 따라 구현한 결과, 문제 없이 네이버 로그인에 성공하여 JWT 토큰까지 생성하는데 성공했다. 그러나 두 가지 문제가 발생했는데 ...

```markdown
[ 문제 1 ] 리액트에서 네이버 로그인을 위해 localhost:8080 으로 전송하면 사용자는 빈 서버 페이지를 보게 된다.

[ 문제 2 ] 스프링에서 발행한 토큰을 리액트로 보낼 방법이 없었다.
```

🔥 위에서 구현한 코드의 문제점을 깨닫고 전면 수정 (성공)

- 스프링에서 자동으로 구현해주는 로직은 내수용(?)인 것으로 추측된다. 즉 스프링 자체만으로 (JSP나 타임리프 같은 템플릿 엔진을 사용) 웹 사이트를 구현한다면 문제가 없겠지만, 우리 프로젝트의 경우 프론트는 리액트, 백은 스프링으로 나눠서 구현했기 때문에 리액트와 통신하며 소셜 로그인을 구현하려면 스프링에서 자동 처리해주는 필터를 사용하면 안 된다.

- 따라서, 아래와 같은 방식으로 개발자가 직접 로그인 로직을 구현해야 한다.

```markdown
1. 리액트에서 네이버 인증 페이지 요청. (사용자가 로그인한다.)

2. 사용자의 로그인이 확인되면 네이버가 인가 코드를 callback url 로 보낸다. (이 때 callback url 은 리액트로 설정)

3. 리액트로 인가 코드를 받으면 스프링으로 fetch 전송한다.

4. 스프링 컨트롤러로 인가 코드가 들어온다.

5. 인가 코드로 네이버에 http 요청해서 접근 코드를 받고, 접근 코드가 들어오면 또 다시 http 요청을 해서 사용자 정보를 받아온다.

6. 사용자 정보로 DB 조회 후, 사용자 정보가 있으면 그대로 진행, 없으면 insert 쿼리를 수행한 후 진행한다.

7. JWT 토큰을 생성하여 리액트로 전송한다.

8. 리액트에서는 fetch 요청의 response 로 들어온 액세스 토큰을 받아 스토리지에 저장한다.
```

🔥 코드로 정리해보자 !

[ 네이버 로그인 요청 ]

- 네이버 인증 url 에 code, client_id, redirect_uri, state 를 파라미터로 넣어서 전송한다.

- redirect_uri 로 네이버 인가 코드가 들어오므로 리액트에서 받도록 설정해줘야 한다.

```javascript
// 네이버 로그인 페이지로 전송
const doNaverLogin = () => {
  let state = encodeURI(process.env.REACT_APP_NAVER_REDIRECT_URI);
  window.location.href =
    "https://nid.naver.com/oauth2.0/authorize?response_type=code" +
    "&client_id=" +
    process.env.REACT_APP_NAVER_CLIENT_ID +
    "&redirect_uri=" +
    process.env.REACT_APP_NAVER_REDIRECT_URI +
    "&state=" +
    state;
};
```

[ 인가 코드 전송 ]

- 사용자가 로그인에 성공하면 callback url 로 code 와 state 값이 파라미터로 들어온다.

- 자바스크립트로 파라미터값을 가져와서 code, state 값을 스프링으로 전송한다. 스프링에서 토큰을 생성하여 http 로 전송해줄 것이고, 이 값은 fetch 요청의 response 로 들어올 것이다.

- response 의 헤더에 접근해 액세스 토큰을 받아 세션 스토리지에 저장한다.

```javascript
// 네이버 로그인 핸들러 (네이버에서 받은 인가코드를 백으로 전송 -> 백에서 인증완료된 JWT 토큰을 받는다)
const urlParams = new URLSearchParams(window.location.search);
const code = urlParams.get("code");
const state = urlParams.get("state");

useEffect(() => {
  if (code !== null) {
    console.log("code?", code !== null);
    handleNaverLogin(code, state);
  }
}, []);

const handleNaverLogin = async (code, state) => {
  const response = await fetch("http://localhost:8080/login/naver", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ code, state }),
  });

  if (response.status === 400) {
    navigate("/");
    alert("사용자 정보가 없습니다. 로그인을 다시 시도해주세요.");
  }

  if (response.status === 200) {
    // accessToken을 세션 스토리지에 저장 (추후 변경 가능성 있음)
    const data = await response.headers.get("Authorization");
    const accessToken = data.split(" ")[1];
    sessionStorage.setItem("accessToken", accessToken);
    navigate("/");
    window.location.reload();
  } else {
    console.error("Failed to fetch token");
  }
};
```

[ 인가 코드로 사용자 정보 획득 ]

- 스프링에서 제공하는 restTemplate 라는 클래스를 사용하면 http 요청을 할 수 있다.

- restTemplate 는 여러 메서드들을 제공하고 있는데, 필요에 따라 사용하면 된다.

| 메서드          | HTTP   | 설명                                                     |
| --------------- | ------ | -------------------------------------------------------- |
| getForObject    | GET    | HTTP GET 요청 후 결과는 객체로 반환                      |
| getForEntity    | GET    | HTTP GET 요청 후 결과는 ResponseEntity 로 반환           |
| postForLocation | POST   | HTTP POST 요청 후 결과는 헤더에 저장된 URL 로 반환       |
| PostForObject   | POST   | HTTP POST 요청 후 결과는 객체로 반환                     |
| PostForEntity   | POST   | HTTP POST 요청 후 결과는 ResponseEntity 로 반환          |
| delete          | DELETE | HTTP DELETE 요청                                         |
| headForHeaders  | HEADER | HTTP HEAD 요청 후 헤더 정보를 반환                       |
| put             | PUT    | HTTP PUT 요청                                            |
| patchForObject  | PATCH  | HTTP PATCH 요청 후 결과는 객체로 반환                    |
| exchange        | Any    | 원하는 HTTP 메서드 요청 후 결과는 ResponseEntity 로 반환 |
| execute         | Any    | Request/Response 의 콜백을 수정                          |

- restTemplate 으로 구현하다보니 자바의 Http 관련 객체들을 다시 공부해보고 싶어 정리해보았다.

- HttpRequest 는 사용자의 요청 정보를 얻기 위해 사용하는 객체이다. 요청의 uri, 메서드(GET or POST 등), 헤더 등을 설정할 수 있다.

- HttpResponse 는 응답을 위해 사용하는 객체이며, 상태 코드, 헤더, 본문을 반환할 수 있다.

- ResponseEntity 는 Spring Framework 에서 사용되는 클래스이며, RESTful API 응답을 좀더 쉽게 구성할 수 있도록 도와준다.

```java
    @Value("${spring.security.oauth2.client.registration.naver.client-id}")
    private String clientId;

    @Value("${spring.security.oauth2.client.registration.naver.client-secret}")
    private String clientSecret;

    /**
     * 네이버 로그인
     * 로그인한 사용자 정보가 디비에 있는지 확인 후 유저 정보 리턴.
     */
    public Authentication getNaverUser(String code, String state) {
        log.info("네이버 로그인 서비스");
        RestTemplate restTemplate = new RestTemplate();

        // 리액트에서 받은 인가 코드로 접근 코드 획득
        String getTokenUrl = "https://nid.naver.com/oauth2.0/token?grant_type=authorization_code"
                + "&client_id=" + clientId + "&client_secret=" + clientSecret
                + "&code=" + code + "&state=" + state;
        Map<String, String> naverToken = restTemplate.getForObject(getTokenUrl, Map.class);
        String naverAccessToken = naverToken.get("access_token");
        log.info("네이버 접근 코드 획득 : {}", naverAccessToken);

        // 접근 코드로 사용자의 로그인 정보 획득
        String getUserUrl = "https://openapi.naver.com/v1/nid/me";
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + naverAccessToken);
        HttpEntity<String> request = new HttpEntity<>(null, headers);
        ResponseEntity<Map> naverResponse = restTemplate.exchange(getUserUrl, HttpMethod.GET, request, Map.class);
        Map<String, String> naverUser = (Map) naverResponse.getBody().get("response");
        log.info("네이버 사용자 정보 획득 : {}", naverUser);

        // 네이버 사용자 정보가 디비에 있는지 확인
        if(naverUser != null) {
            String username = naverUser.get("nickname");
            String email = naverUser.get("email");
            String role = "USER";

            User user = userMapper.findByUsername(username);

            if(user == null) {
                user = User.builder()
                        .nickname(username)
                        .email(email)
                        .platform("naver")
                        .roles(role)
                        .build();
                userMapper.insertOauthUser(user);
                log.info("새로운 oauth2 유저를 등록했습니다 : {}", user);
            }

            Authentication authentication = new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities());
            SecurityContextHolder.getContext().setAuthentication(authentication);
            log.info("oauth2 유저가 인증되었습니다 = {}", authentication);

            return authentication;

        } else {
            log.info("네이버 사용자의 정보를 확인할 수 없습니다.");
            return null;
        }
    }
```

구현에 성공하고 보니 간단한 로직을 내가 괜히 어렵게 생각했구나 하는 생각이 들었다. 네이버 공식 문서가 설명이 잘 되어 있어서 그대로만 따라해도 문제가 없었다. 깃허브 로그인도 마저 구현해야 하지만 일단 메인페이지를 먼저 구현하기로 했다.