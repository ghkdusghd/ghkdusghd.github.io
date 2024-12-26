---
title: "[NaverCloud] HTTPS 통신을 위한 SSL 인증서 발급하기"
parent: NCBT
nav_order: -10

tags:
  - [ncp, ssl]

toc: true
toc_sticky: true

date: 2024-12-23
last_modified_at: 2024-12-23
---

> 백, 프론트 서버 모두 배포가 완료되어 기능을 테스트해보았으나 혼합 콘텐츠 오류가 발생했다. Vercel 로 배포한 웹 페이지는 HTTPS 로 로드되었지만, 그 페이지 내에서 HTTP 를 사용하여 리소스를 요청하려고 시도했기 때문에 보안을 위해 해당 요청을 차단하는 것이다. 이를 해결하기 위해서 리액트와 스프링 서버 간 HTTPS 로 통신할 수 있도록 SSL 인증서를 발급하기로 했다.

### 1️⃣ 인증서 발급 기관 선택

> #### 네이버 클라우드 Certificate Manager (❌)

처음에는 네이버 클라우드에 certificate manager 라는 인증서 발급 기능이 있길래 그걸로 하려고 했다. 네이버 클라우드 서버를 이용중이라면 싱글 도메인은 무료로 발급할 수 있지만, 알고보니 연동된 서비스 (로드 밸런서, CDN 등) 에서만 사용할 수 있었고, Advanced 인증서를 발급받으려면 비용이 발생하는 구조였다. 요금은 28,000원... 부담스러운 금액은 아니지만 무료로도 좋은 서비스가 있지 않을까 싶어 찾아보았다.

> #### Let's Encrypt (✅)

HTTPS 의 확산을 위해 무료 인증서 보급을 시작한 비영리 프로젝트라고 한다. 아파치 서버에서는 완전 자동으로 SSL 인증을 획득하고 적용할 수 있고, 인증 기간은 90일로 상대적으로 짧지만 자동 갱신 기능을 지원하고 있어 간편하다. 이걸로 인증서를 발급해보기로 했다.

아, 물론 도메인은 따로 사야 한다. 우리는 가비아에서 구매했다.

### 2️⃣ Let's Encrypt 인증서 생성하기

> #### 1단계 : Apache 설치

우선 네이버 클라우드 서버의 root 계정으로 접속한다. Apache 웹 서버가 없다면 설치해야 한다.

``` bash
sudo apt update
sudo apt install apache2
```

<br>

> #### 2단계 : Certbot 설치

Let's Encrypt 인증서는 Certbot 을 통해 발급받을 수 있다. 

``` bash
sudo apt update
sudo apt install certbot python3-certbot-apache
```

<br>

> #### 3단계 : ACG 에서 80포트(HTTP), 443포트(HTTPS) 오픈

Apache 가 HTTP, HTTPS 포트에서 정상적으로 작동하도록 ACG 인바운드 설정을 변경해주어야 한다.

<img src="/assets/images/pages/projects/ncbt/스크린샷 2024-12-24 오후 1.00.19.png" />

<br>

> #### 4단계 : 인증서 발급

이 단계에서 실패한다면 도메인의 포워딩 설정을 다시 체크하자. 우리가 구매한 도메인이 서버의 공인 IP 주소로 포워딩 되도록 만들어야 한다.

``` bash
sudo certbot --apache
```

<br>

🔖 참고문서

[Let's Encrypt 인증서 발급 및 갱신법](https://www.owl-dev.me/blog/42)