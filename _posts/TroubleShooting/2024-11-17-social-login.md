---
title: "[트러블슈팅] OAuth2 로그인 토큰 전송 오류"
excerpt: "-"

writer: Hwayeon Hong
categories:
  - TroubleShooting
permalink: /troubleShooting/:title/
tags:
  - [oauth2, naver]

toc: true
toc_sticky: true

date: 2024-11-17
last_modified_at: 2024-11-17
---

> oauth 로그인을 구현하면서 겪은 시행착오를 정리해본다.

### 🔥 문제 상황

- 네이버 로그인이 완료되면 우리 서버에서 JWT 토큰을 생성해서 클라이언트에게 전송하는데, 액세스 토큰은 http 헤더로, 리프레쉬 토큰은 http only 쿠키로 전송한다. 그런데 클라이언트에서 인증 헤더에 접근하지 못하는 문제가 발생했다.

### ✨ 해결

- 스프링 서버에서 response 로 받은 값을 콘솔에 찍어보았는데 헤더에 있어야 할 토큰값이 안보였고, 여기서 type="cors" 라는 텍스트가 보여서 이 부분이 뭔가 쎄하긴 했지만 계속 시도해보았다. 그래도 잘 되지 않았음.

<img width="778" alt="스크린샷 2024-10-17 오전 11 47 32" src="https://github.com/user-attachments/assets/71bc7020-af2b-45cd-979d-ce97dc964546">

- 여러 번 시도해봐도 헤더에 접근할 수 없어서 구글링한 결과, cors 에러가 원인인 것 같았다. cors 때문에 헤더에 접근하는 것에 제한이 걸리고, 추가적인 설정을 해주지 않으면 몇몇 standard 한 값만 보이는 것이다... 그래서 cors 설정을 추가했다. config.addExposedHeader("Authorization"); 단 한 줄 추가로 해결했다.

- cors 관련 설정은 초기에 설정해두었고, 이후로도 별 문제가 없어서 관련 에러는 다 잡았다고 생각했는데 뒷통수 맞은 느낌이었다... 방심하면 안 되는 코딩생활...
