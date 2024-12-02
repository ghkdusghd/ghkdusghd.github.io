---
title: "[OAuth2] react + spring boot 로 깃허브 소셜 로그인 구현"
excerpt: "-"

writer: Hwayeon Hong
# categories:
#   - projects
tags:
  - [oauth2, github]

toc: true
toc_sticky: true

date: 2024-12-02
last_modified_at: 2024-12-02
---

---

> 네이버에 이어서 깃허브 소셜 로그인 구현에 성공했다. 깃허브는 네이버에 비하면 공식문서가 부실해서 예상치 못한 부분에서 헤매야 했다 ㅠㅠ 나중을 위해 구현하면서 알게 된 내용들을 정리해보고자 한다.

<br>

### 1️⃣ Oauth2 로그인 과정 (깃허브)

```markdown
1. 리액트에서 깃허브 API 로 접근코드(code) 요청
2. 깃허브에서 client_id 인증 후 접근코드를 리액트로 반환
3. 리액트에서 반환받은 접근코드를 스프링 부트 서버로 전송
4. 서버에서 접근코드로 깃허브 API 에 요청하여 access token 을 반환받음
5. 반환받은 access token 으로 다시 깃허브 API 에 사용자 정보 요청
6. 반환받은 사용자 정보로 DB 조회 및 토큰 발행 등, 필요한 서비스 로직 처리
7. 로그인 성공
```

<br>

### 2️⃣ 깃허브 로그인 특이사항

- access token 요청 시 POST 메서드로 요청하지만, client_id 등의 필수 정보는 body 가 아닌 파라미터로 전송해야 한다.

  - 나의 http 관련 지식이 부족해서겠지만, POST 요청 시에는 무조건 body 에 데이터를 세팅해서 보내야 한다고 생각했다. 그래서 계속 body 를 사용하여 파라미터들을 전송했는데, 알고보니 queryParam 으로 보내야 했다.

- 이메일 정보는 따로 요청해야 한다.

  - 기본적인 공개 정보는 사용자 프로필 정보, 리포지토리 정보, gist 정보이다. 이외로 필요한 정보가 있다면 따로 요청해야 한다. 우리 서비스의 경우 이메일 정보가 필요했기에 추가로 요청했다.

  | No. | 순서                                                      | 해설                                                                             |
  | --- | --------------------------------------------------------- | -------------------------------------------------------------------------------- |
  | 1   | 리액트에서 접근코드 요청 시 scope=user:email 을 추가한다. | https://github.com/login/oauth/authorize?client_id={MyClientId}&scope=user:email |
  | 2   | 스프링 서버에서 이메일 정보를 추가로 요청한다.            | https://api.github.com/user/emails 경로로 GET 요청                               |
  | 3   | 이메일 정보는 List 형태로 받는다.                         | 깃허브는 사용자가 여러 개의 이메일을 설정할 수 있기에 배열 형태로 받는 것이다.   |

<br>

[ 🚀 코드로 정리해보자 ]

전체적인 oauth2 로그인 동작 코드는 [이전 글 (클릭하면 이동)](https://ghkdusghd.github.io/2024/11/17/naver-login.html) 에 정리해두었으니 이번 글에서는 헤맸던 부분들만 정리해보려고 한다.

- POST 요청 및 param 세팅 : 자바에서 쿼리 등 uri 구성 요소들을 어떻게 넣을까 싶었는데 스프링에서 URI 빌더 클래스를 제공하고 있었다. 여러 개의 query 가 들어가게 되면 코드가 번잡해지는데, key value 형식으로 파라미터를 넣어줄 수 있어서 보기도 쉽고 간편하게 생성할 수 있었다.

- 요청 헤더 설정 : 깃허브 공식문서를 보면 기본적으로 access token 은 아래와 같은 쿼리 파라미터 형식으로 반환하는데, 나는 객체 타입으로 받고 싶어서 ccept: application/json 헤더를 추가했다.

```
access_token=gho_16C7e42F292c6912E7710c838347Ae178B4a&scope=repo%2Cgist&token_type=bearer
```

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://github.com/login/oauth/access_token")
        .queryParam("client_id", githubClientId)
        .queryParam("client_secret", githubClientSecret)
        .queryParam("code", code)
        .encode()
        .build().toUri();
RequestEntity<Void> requestEntity =
        RequestEntity.post(uri)
                .header("Accept", "application/json").build();
```

<br>

- 사용자 이메일 정보 요청하기

  - 위에서도 간단히 설명했지만, 깃허브는 이메일 정보까지 받고 싶으면 scope 를 추가해줘야 한다. 이 scope 는 리액트에서 접근코드를 요청할 때 같이 넣어주면 된다.

  ```javascript
  const doGitLogin = () => {
    window.location.href =
      "https://github.com/login/oauth/authorize?client_id=" +
      process.env.REACT_APP_GITHUB_CLIENT_ID +
      "&scope=user:email";
  };
  ```

  - 그리고 반환받은 code 를 스프링 서버로 전송한다. 이 과정은 네이버 로그인을 구현했을 때와 동일하다.

  ```javascript
  const response = await fetch("http://localhost:8080/login/github", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ code }),
  });
  ```

  - 서버에서 사용자 정보를 요청할 때는 https://api.github.com/user 로, 이메일 정보는 https://api.github.com/user/emails 경로로 요청하면 된다.

  - 주의해야 할 점은, 사용자 한 명이 여러 개의 이메일 정보를 가지고 있을 수 있으므로 List 형태로 반환받을 것.

  ```java
  // access token 을 사용하여 사용자의 로그인 정보 획득
  String getUserUrl = "https://api.github.com/user";
  HttpHeaders headers = new HttpHeaders();
  headers.set("Authorization", "Bearer " + githubAccessToken);
  HttpEntity<String> requestUser = new HttpEntity<>(null, headers);
  ResponseEntity<GithubUserDTO> githubResponse = restTemplate.exchange(getUserUrl, HttpMethod.GET, requestUser, GithubUserDTO.class);
  GithubUserDTO githubUser = githubResponse.getBody();

  // 깃허브는 이메일 정보는 따로 요청해야 한다...
  String getEmailUrl = "https://api.github.com/user/emails";
  HttpEntity<String> requestEmail = new HttpEntity<>(null, headers);
  ResponseEntity<List<GithubEmailDTO>> emailResponse = restTemplate.exchange(getEmailUrl, HttpMethod.GET, requestEmail, new ParameterizedTypeReference<List<GithubEmailDTO>>() {});
  List<GithubEmailDTO>  emails = emailResponse.getBody();
  String primaryEmail = emails.isEmpty() ? "이메일 없음" : emails.get(0).getEmail();
  ```

- 이메일 형식이 뭔가 이상해도 어쩔 수 없다 !

  - 처음 깃허브 소셜로그인 구현에 성공했을 때, 도메인이 gmail.com 이나 naver.com 같은 익숙한 구조가 아니라 당황했다. 그래서 내가 깃허브에 등록한 이메일 정보를 보니, 'Keep my email addresses private' 라는 기능이 활성화되어 있었다.

  - 해당 기능을 활성화한 유저의 이메일은 '숫자 + 깃허브 아이디 @ 임의 도메인' 으로 구성되어서 본인이 아니고서는 실제 이메일을 확인할 수 없게 되어 있다.

  - 우리 서비스에서 이메일의 용도는 실제 메일을 발송하기 위한 것이 아닌, 중복 가입을 방지하기 위한 것이라 고유한 이메일이라면 문제가 없어서 이대로 저장하기로 결정했다.

<br>

### 🤓 배운 점

이미 네이버 로그인을 구현한 상태라 쉽게 완성할 수 있을 것이라 생각했는데, 생각 외로 여러모로 공부가 많이 되었다.

- <b>자바에서 URI 클래스를 처음 사용해보았다.</b>
  <br>
  나는 리액트 + 스프링 으로 구성된 프로젝트만 경험해보았기 때문에 uri 요청은 전부 리액트를 통해서 진행했었기에 자바에서는 이런 식으로 http 요청을 하면 되는구나를 몸으로 배웠다.

- <b>RestTemplate 클래스를 처음 사용해보았다.</b>
  <br>
  마찬가지로, 자바에서 http 요청을 하기 위한 클래스를 처음 사용해 보았다. 이번 깃허브 소셜로그인을 구현하면서 새로 알게 된 점은 restTemplate 의 exchange 메서드에서는 제네릭을 인자로 넣으면 타입을 인식하지 못해 LinkedHashMap 으로 자동으로 형변환 된다는 점이었다.
  <br>
  이에 대한 해결책으로 ParameterizedTypeReference 라는 클래스가 있다고 하여 사용해보았다. 제네릭 타입을 잡아 런타임에 유지시켜주는 추상클래스라고 한다.

이제 남은 작업은 토큰 갱신이다. 이전에 JWT토큰 로직을 구현하면서 인증, 생성 부분만 완성하고 갱신 로직은 뒤로 미뤄뒀었는데, 프로젝트가 마무리되어가는 시점이라 빠르게 완성을 해야 할 것 같다...

<br>

🔖 참고자료

[(블로그) Spring Boot Github 소셜 로그인 구현하기](https://inkyu-yoon.github.io/docs/Language/SpringBoot/GithubLogin)

[Github Docs - Authorization](https://docs.github.com/ko/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps#web-application-flow)

[Github Docs - scope](https://docs.github.com/ko/apps/oauth-apps/building-oauth-apps/scopes-for-oauth-apps)
