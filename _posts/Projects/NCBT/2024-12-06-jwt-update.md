---
title: "[Spring Security + JWT] 토큰 재발급을 통한 로그인 유지"
excerpt: "-"

writer: Hwayeon Hong
# categories:
#   - projects
tags:
  - [spring security, jwt]

toc: true
toc_sticky: true

date: 2024-12-06
last_modified_at: 2024-12-06
---

---

> 이번에는 JWT 토큰을 활용하여 로그인을 유지하는 로직을 구현해 보았는데, 간단한 듯 보이면서도 고려해야 할 점이 많았다. 여러 경우의 수가 있을 수 있기 때문에 흐름도를 작성해보면 도움이 될 것 같아서 나름대로 도식화를 해보았고, 그 과정에서 기존에 작성해둔 토큰 발급 코드도 수정하게 되었다.

### 1️⃣ JWT 토큰 발급 순서도

##### 🚀 토큰 발급 전

<img width="931" alt="스크린샷 2024-12-09 오후 12 36 39" src="https://github.com/user-attachments/assets/cd449008-da50-4279-ac93-aad39267e911">

##### 💫 토큰 발급 후

<img width="961" alt="스크린샷 2024-12-09 오후 12 37 06" src="https://github.com/user-attachments/assets/e5fd770e-00ee-47db-8826-fdacdce001c8">

### 2️⃣ 구현 과정

### Front-End

<b>Nav.jsx</b>
<br>
Nav 바는 NCBT 서비스의 모든 페이지에 띄워지기 때문에, 처음 마운트 되었을 때 access token 을 확인하도록 했다.

    - 토큰 만료 확인 : access token 의 기한이 만료되었다면 세션 스토리지에서 삭제한다.

    - 토큰 재발급을 요청 : access token 이 없는 상태라면 refresh token 을 서버로 전송하여 토큰 재발급을 요청한다.

### Back-End

<b>"/form/login", "/login/naver", "/login/github" API</b>
<br>
폼 로그인 및 소셜 로그인 요청 API

    - 해당 API 로 들어오는 요청은 시큐리티 필터를 거치지 않도록 예외처리 한다.
    - 요청으로 들어온 사용자 정보와 DB 에 저장된 사용자 정보가 일치하면 access token, refresh token 을 생성한다.
    - access token => 응답 헤더로 전송하고, 프론트에서 세션 스토리지에 저장한다.
    - refreshs token => http only 쿠키로 전송한다. 또한, DB 에 토큰의 만료 시간과 상태를 저장하여 사용자의 로그인 유지를 위해 관리한다.

<b>"/refreshToken" API</b>
<br>
토큰 재발급 요청 API

    [JwtAuthFilter]
    - 해당 API 로 토큰 갱신 요청이 들어오면 시큐리티 필터 체인에서 access token, refresh token 이 유효한지 확인한다.
    - 유효하다면 인증 객체 (Authenticataion) 을 생성하고, 유효하지 않다면 401 상태코드를 반환한다.

    [Controller]
    - access token, refresh token 을 재발급하여 각각 response header 와 http only 쿠키로 전송한다.
    - 이 때 사용한 refreshs token 은 USED 상태로 변경한다. (UPDATE 쿼리문 실행)

<b>"/form/logout" API</b>
<br>
로그아웃 API

    - 해당 API 로 들어오는 요청은 시큐리티 필터를 거치지 않도록 예외처리 한다.
    - DB 에 저장된 refresh token 의 상태를 EXPIRED 로 변경한다. (UPDATE 쿼리문 수행)
    - 쿠키의 만료시간을 0으로 변경하는 것으로 삭제한다.

### 🤓 배운 점

<br>
1. React 상태관리

    로그인 유지를 위해서는 재접속한 사용자의 토큰이 남아 있는지를 확인해야 해서 우리 사이트에 항상 띄워져있는 Nav 바에서 토큰을 확인하도록 하고, useEffect 를 활용해 사용자가 접속하여 페이지를 옮길 때마다 실행하도록 만들었는데...

    [문제 1] 한 번 재발급이 성공했음에도 불구하고 계속해서 여러번, 무한으로 렌더링 되는 오류가 발생했다.

    [문제 2] 사용자가 로그인한 이후로 브라우저를 종료한 후 재접속한 경우... 이 경우에는 리프레쉬 토큰이 남아 있음에도 불구하고 토큰 재발급 로직이 실행되지 않았다.

    [해결] 찬찬히 확인해보니 useEffect 의 의존성 배열이 문제였다. Nav 바의 로직은 기존에 만들어둔 로직을 현재의 요구사항에 맞춰 수정한 것이라서 의존성 배열 안에 [token] 이라는 값이 들어가있는 상태였다. 즉 액세스 토큰에 변동이 있을 때마다 useEffect 가 실행되도록 만들어놓아서 어떤 페이지에서는 실행되지 않거나, 실행된 이후에 무한으로 실행되는 문제가 발생했던 것이었다. 그래서 의존성 배열을 [] 로 없애 마운트할 때만 토큰 관련 함수가 실행되도록 바꾸는 것으로 문제를 해결할 수 있었다.

<br>
2. 로그를 한땀 한땀 촘촘하게 찍자 !

    토큰 재발급 로직을 구현하면서, 이미 만들어둔 로직에 살을 붙여가는 형태로 개발을 했는데, 그렇다보니 (내가 작성한 코드임에도 불구하고...) 요청이 들어왔을 때 어떤 식으로 흘러가는지 알기 어려웠다. 그래서 아주 세세하게 로그를 찍기 시작했다.

    log.info("1");
    log.info("2");
    log.info("3");

    이런 식으로 한 줄마다 로그를 찍어보니 요청이 어떻게 흘러가고 어디서 막히는지 손쉽게 확인할 수 있었고, 어떤 부분에서 발생한 예외인지 한눈에 보여서 빠르게 해결책을 찾을 수 있었다.
