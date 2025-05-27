---
title: "[트러블슈팅] 깃허브 소셜로그인 bad credentials 해결"
parent: OAuth
nav_order: -3

date: 2025-05-27
---

# <span style="color: #7153ED; font-weight: bold;">트러블슈팅 </span> 깃허브 소셜로그인 bad credentials 해결

## 📌 이슈

스프링에서 깃허브로 사용자 정보를 요청했으나 401 Unauthorized 응답을 받음.

``` java
org.springframework.web.client.HttpClientErrorException$Unauthorized: 401 Unauthorized: "{"message":"Bad credentials","documentation_url":"https://docs.github.com/rest","status":"401"}"
```

## 👩🏻‍💻 해결 과정 


<b>1️⃣ 스프링의 RestClient 요청은 총 세 번 이루어진다. 어느 시점에 에러가 발생했는지 확인하기 위해 로그를 찍었다.</b>

``` java

2025-05-27T13:40:13.181+09:00  INFO 22864 --- [backend] [nio-8080-exec-3] k.kh.backend.security.jwt.JwtAuthFilter  : httpUri = /login/github
2025-05-27T13:40:13.208+09:00  INFO 22864 --- [backend] [nio-8080-exec-3] k.kh.backend.controller.UserController   : 깃허브 로그인 컨트롤러
2025-05-27T13:40:13.208+09:00  INFO 22864 --- [backend] [nio-8080-exec-3] k.k.b.s.security.Oauth2UserService       : 깃허브 로그인 서비스
2025-05-27T13:40:13.208+09:00  INFO 22864 --- [backend] [nio-8080-exec-3] k.k.b.s.security.Oauth2UserService       : request access token
2025-05-27T13:40:13.605+09:00  INFO 22864 --- [backend] [nio-8080-exec-3] k.k.b.s.security.Oauth2UserService       : get access token
2025-05-27T13:40:13.606+09:00  INFO 22864 --- [backend] [nio-8080-exec-3] k.k.b.s.security.Oauth2UserService       : request user info
org.springframework.web.client.HttpClientErrorException$Unauthorized: 401 Unauthorized: "{"message":"Bad credentials","documentation_url":"https://docs.github.com/rest","status":"401"}"
```

깃허브 로그인은 다음과 같은 방식으로 진행된다. 우선 스프링에서 접근 코드를 보내고 인증이 완료되면 access token 을 응답으로 받는다. 해당 access token 으로 다시 깃허브에 요청하여 사용자 정보를 응답으로 받는다. 위 로그를 보면 access token 을 응답받는 부분까지는 문제가 없었고,(즉 접근 코드는 문제가 없다) 사용자 정보를 요청하는 부분에서 401 에러가 발생했다는 것을 알 수 있다.

<br>

<b>2️⃣ Github Docs에 접속하여 에러 관련 내용을 확인해보니 아래 내용이 눈에 띄었다.</b>

[Github Docs 이동하기](https://docs.github.com/ko/rest/using-the-rest-api/troubleshooting-the-rest-api?apiVersion=2022-11-28#404-not-found-for-an-existing-resource)

``` markdown
- GitHub App 설치 액세스 토큰을 사용하는 경우 다음을 확인해야 합니다.
- OAuth App 사용자 액세스 토큰을 사용하는 경우 다음을 확인해야 합니다.
```

Github App과 OAuth App 두 개가 있다는 사실을 깨달았다. 사실 NCBT 프로젝트를 새 버전으로 재설계하던 와중에 이번 이슈가 발생한 것이라서 내가 기존에 어떤 Apps 를 선택했는지 기억하지 못했고, Developer Settings 에서 Github App 항목에 NCBT 항목이 있어서 그걸 사용한 것이었다. 위 내용을 보고 다시 확인해보니 놀랍게도 두 항목에 전부 NCBT 라는 이름의 클라이언트 ID 를 만들어둔것을 발견했다. 내가 만들려는 기능은 사용자 인증 및 권한 위임을 받기 위한 것이므로 OAuth App 으로 만든 클라이언트 ID 를 사용하도록 application.yml 및 .env 파일을 변경했다.

어쩐지 이 부분에서 평소 보던 화면과 다른 것 같은 묘한 이질감이 있었는데 완전 다른 클라이언트 ID 로 요청해서 그런거였다..

<br>

<b>3️⃣ 깃허브 클라이언트 ID를 OAuth App 항목의 ID로 변경하니 소셜로그인 로직이 정상적으로 처리되었다.</b>

``` java
2025-05-27T14:14:19.998+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.kh.backend.security.jwt.JwtAuthFilter  : httpUri = /login/github
2025-05-27T14:14:20.006+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.kh.backend.controller.UserController   : 깃허브 로그인 컨트롤러
2025-05-27T14:14:20.007+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.k.b.s.security.Oauth2UserService       : 깃허브 로그인 서비스
2025-05-27T14:14:20.007+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.k.b.s.security.Oauth2UserService       : request access token
2025-05-27T14:14:20.539+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.k.b.s.security.Oauth2UserService       : get access token
2025-05-27T14:14:20.539+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.k.b.s.security.Oauth2UserService       : request user info
2025-05-27T14:14:21.216+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.k.b.s.security.Oauth2UserService       : get user info
2025-05-27T14:14:21.236+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.k.b.s.security.Oauth2UserService       : oauth2 유저가 인증되었습니다
2025-05-27T14:14:21.245+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.k.b.security.jwt.JwtTokenProvider      : Generate JWT token
2025-05-27T14:14:21.246+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.k.b.security.jwt.JwtTokenProvider      : user roles = USER
2025-05-27T14:14:21.290+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.k.b.security.jwt.JwtTokenProvider      : generated access Token
2025-05-27T14:14:21.291+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.k.b.security.jwt.JwtTokenProvider      : generated refresh Token
2025-05-27T14:14:21.370+09:00  INFO 40418 --- [backend] [nio-8080-exec-3] k.k.b.security.jwt.JwtTokenProvider      : 리프레시 토큰 저장 완료
```

<br>

<b>4️⃣ 토큰이 문제 없이 저장된 것도 확인한다.</b>

<img src="/assets/images/pages/TroubleShooting/스크린샷 2025-05-27 오후 2.21.13.png">

## 💡 느낀점 및 기억할 정보

이슈를 빠르게 해결하기 위한 방법은 단계적으로 확인해보는 일 같다. 마음이 급하면 구글링을 먼저 하거나 GPT 에 물어보는 일을 반복하면서 삽질을 하게 되는데, 막상 내가 만난 이슈에 딱 맞는 해결책은 쉽게 찾기 어렵다. 이럴 때는 차분한 마음으로 안 되는 부분이 어디인지를 먼저 찾고, 이번처럼 힌트가 주어진 경우 그 부분을 먼저 찾아보는게 제일 빠르다. 이번 건도 GPT 에 계속 물어봤는데 엉뚱한 해답뿐이라 답답했는데, 해결 방법을 알고 보니 맹점이었고 내가 눈치채지 못하면 찾기 힘든 방법이었다. 차근차근 원인을 분석해 나가면 언젠가 해결할 수 있다는 것을 알고 습관화하면 좋을 것 같다.