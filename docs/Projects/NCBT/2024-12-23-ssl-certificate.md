---
title: "[NaverCloud] HTTPS 통신을 위한 SSL 인증서 발급하기"
parent: NCBT
nav_order: 10

tags:
  - [ncp, ssl]

toc: true
toc_sticky: true

date: 2024-12-23
last_modified_at: 2024-12-23
---

> 백, 프론트 서버 모두 배포가 완료되어 기능을 테스트해보았으나 혼합 콘텐츠 오류가 발생했다. Vercel 로 배포한 웹 페이지는 HTTPS 로 로드되었지만, 그 페이지 내에서 HTTP 를 사용하여 리소스를 요청하려고 시도했기 때문에 보안을 위해 해당 요청을 차단하는 것이다. 이를 해결하기 위해서 리액트와 스프링 서버 간 HTTPS 로 통신할 수 있도록 SSL 인증서를 발급하기로 했다.

> ### 1️⃣ 네이버 클라우드 Certificate Manager

SSL 인증서를 손쉽게 등록, 발급, 관리할 수 있는 서비스이다. 보통 인증서 발급은 비용이 들지만, 네이버 클라우드 서버를 계약했다면 싱글 도메인 인증서는 무료로 발급할 수 있어서 선택했다.

아, 그리고 도메인은 따로 구매해야 한다. 우리는 가비아에서 구매했다.

#### 1단계 : [인증서 발급 신청(클릭하면 이동)](https://console.ncloud.com/certificate-manager/management)

- Cloud Basic 무료 인증서를 선택하자. 인증방식은 DNS 인증을 선택했다.

<img width="1213" alt="스크린샷 2024-12-23 오후 3 36 40" src="https://github.com/user-attachments/assets/dab82162-6757-40ad-866a-994602c66212" />

<img width="1230" alt="스크린샷 2024-12-23 오후 3 38 31" src="https://github.com/user-attachments/assets/185a091e-e895-4087-900c-31aa1467c3c2" />

<img width="1231" alt="스크린샷 2024-12-23 오후 3 40 20" src="https://github.com/user-attachments/assets/c6f8c2f2-3021-4f31-ad86-1c66c7700acd" />

#### 2단계 : 도메인 CNAME 추가

- 이후 도메인의 소유주를 검증하기 위한 절차를 진행한다. 인증서 발급신청이 완료되면 Record Name, Record Values 를 제공하는데, 이걸 도메인을 구매한 사이트의 CNAME 으로 등록하면 된다.

- 검증 절차는 시간이 걸린다.