---
title: "[트러블슈팅] HTTP Only 쿠키에 Refresh Token 을 저장해보자"
parent: OAuth
nav_order: -2

tags:
  - [cookie, http]

toc: true
toc_sticky: true

date: 2024-12-04
last_modified_at: 2024-12-04
---

우리 NCBT 프로젝트는 Refresh Token 을 저장하기 위한 방법으로 쿠키를 선택했다. 쿠키 외의 선택지는 세션 스토리지 정도가 있었는데, 세션은 브라우저가 종료되면 저장해둔 데이터가 사라지므로 액세스 토큰 재발급 용도의 토큰을 세션에 저장하는 것은 적절치 않다고 생각했고, 또한 세션이나 로컬 스토리지에 저장한 데이터는 자바스크립트로 조작이 가능하기 때문에 XSS (Cross Site Scripting) 공격에 취약하다고 하여 쿠키로 선정하게 되었다.

<br>
그래서 쿠키에 대해 공부하면서 프로젝트에 직접 적용해보았는데, 스프링 서버에서 전송한 쿠키가 브라우저 애플리케이션 탭에 저장되지 않아 하루 종일 삽질을 했다... 결론부터 말하자면 단 한줄로 해결이 되었지만, 이러한 일을 겪으면서 쿠키에 대해 공부한 내용을 정리해 보고자 한다.

<br>

### 🍪 '쿠키' 란 ?

오해하면 안 되는 점이 쿠키 역시 세션 or 로컬 스토리지와 마찬가지로 자바스크립트로 접근이 가능하지만, 'HTTP Only' 옵션을 걸어주면 이를 막을 수 있다고 한다. 다만, 쿠키는 CSRF 공격에는 취약하다고 하는데... XSS 공격은 `쿠키에 저장된 토큰 값` 자체를 탈취하는 것이고, CSRF 는 `로그인 된 상태로 위험한 동작을 하게 만드는` 공격이라고 보면 된다.

<br>
그럼에도 불구하고, refresh token 은 access token 을 재발급 해주는 역할 외에는 인증, 인가 로직을 통과할 수 없기에 (왜냐면 권한 정보가 없기 때문이다) 쿠키에 저장하기로 했다.

<br>

### 🍪 쿠키 전송 옵션 및 주의사항

쿠키를 생성하고, 전송할 때의 옵션은 여러가지가 있다. 각자의 상황에 맞게 생성하여 전송하면 된다. 우선 우리 서비스는 아직 배포 전 개발단계에 있기 때문에 localhost 도메인을 사용하고 있었고, 당연히 https 통신은 할 수 없는 상황이었다. 그래서 아래와 같은 방식으로 쿠키를 생성했다.

<br>

```java
Cookie refreshCookie = new Cookie("refreshToken", jwtToken.getRefreshToken());
refreshCookie.setHttpOnly(true); // HttpOnly 설정
refreshCookie.setMaxAge(24 * 60 * 60); // 쿠키 유효기한 (1일)
refreshCookie.setSecure(false); // secure 를 true 로 설정하면 https 통신에서만 쿠키를 보낼 수 있다.
refreshCookie.setPath("/"); // 쿠키가 유효한 경로를 설정한다. "/" 루트 경로로 설정하게 되면 모든 경로에서 이 쿠키가 유효하게 된다.
response.addCookie(refreshCookie);

return ResponseEntity.ok()
        .header("Authorization", "Bearer " + jwtToken.getAccessToken())
        .header("Set-Cookie", "refreshToken=" + jwtToken.getRefreshToken()
                + "; Path=/; HttpOnly; Max-Age=86400; SameSite=Lax" )
        .build();
```

<br>

### 🍪 SameSite = Lax ?

SameSite 속성은 CSRF 공격을 방지하는 데 사용된다고 한다. Lax 는 쿠키가 간단한 탐색 (링크 클릭 등) 에서만 전송되도록 제한하고, Strict 로 설정하면 더 엄격한 설정이 가능하다고 한다. 다만, Strict 로 설정하게 되면 서로 다른 도메인에서는 아예 전송이 불가능해진다. 이 경우 반드시 Secure 를 true 로 설정해야 하는데... 우리 서비스는 아직 개발 중이라 Lax 로 설정하게 되었다.

<br>

### 🤓 트러블슈팅

#### before

이제 본론으로... 내가 겪은 문제는 서버에서 쿠키가 전송이 되지만 브라우저에 저장이 되지 않는다는 점이었다. 억울한 점은 [네트워크] 탭에서는 쿠키가 전송이 되고 있다는 것인데, 여기서 내가 간과한 점은 '쿠키' 라는 것이 서버에서 http 통신으로 전송만 해주면 자동으로 브라우저에 저장이 된다고 생각했다는 점이다...

<img width="1437" alt="스크린샷 2024-12-09 오후 2 24 12" src="https://github.com/user-attachments/assets/a3747f83-0011-4af1-8dc1-e7918648645e">

#### after

그래서 쿠키로 개발한 경험이 있었던 팀원에게 이 문제를 상담했는데, 리액트에서 응답으로 들어온 쿠키를 받아서 세팅까지 해줘야 한다는 점을 알게 되었다. 아래의 단 한줄의 코드가 필요했던 것...

```javascript
response.headers.get("Set-Cookie");
```

<img width="1304" alt="스크린샷 2024-12-09 오후 2 27 03" src="https://github.com/user-attachments/assets/9f1ad91f-0f81-40fd-aad8-bcaf049f874f">

<br>

#### 배운 점

쿠키가 자동으로 저장이 된다? 라고 생각했던 이유는 단순하게도... HTTP Only 쿠키는 자바스크립트로 조작할 수 없다고 했기에, 그렇다면 리액트에서 콘솔에 찍어서 확인하거나 저장하는 조작도 전부 할 수 없는 것이 아닌가! 라고 자의적인 판단을 해버린 것이 원인이다. 잘 알지 못하는 기술에 대해서 안이하게 판단해 버렸기 때문에 하루라는 시간을 날리게 되었다...

<br>

🔖 참고자료

[(블로그) Refresh Token 을 어디에 저장할까](https://velog.io/@ohzzi/Access-Token%EA%B3%BC-Refresh-Token%EC%9D%84-%EC%96%B4%EB%94%94%EC%97%90-%EC%A0%80%EC%9E%A5%ED%95%B4%EC%95%BC-%ED%95%A0%EA%B9%8C)