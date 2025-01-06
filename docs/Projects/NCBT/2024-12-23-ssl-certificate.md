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

우선 네이버 클라우드 서버의 root 계정으로 접속한다. Apache 웹 서버가 없다면 설치해야 한다. 이 때, 스프링 컨테이너가 실행중이라면 ```docker stop 컨테이너명``` 으로 일단 종료하자 !

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

``` bash
# 인증서 발급 시작
# 이메일 입력, HTTP/HTTPS 리디렉션 설정 여부, 서비스 약관 동의 등의 내용이 나온다.
# A 로 전부 동의해주고 넘어가기
sudo certbot --apache
```

<br>

> #### 🤯 인증서 발급 실패 !?

구매한 도메인의 포워딩 설정을 확인하자. 우리는 ncbt.site 라는 도메인을 구매했고, 루트 도메인과 서브 도메인으로 나눠 사용할 것이다. 보통 도메인에 www 가 붙으면 루트 도메인으로 인식된다. 우리는 여기에 프론트 서버의 공인IP 를 연결했다. 그리고 api 라는 서브 도메인을 백 서버의 공인IP 로 포워딩되게 만들었다. 아래와 같이 가비아에서 설정해주면 된다.

| TYPE | NAME | CONTENT |
|---|---|---|
| A | @ | 프론트 서버의 공인 IP |
| A | www | 프론트 서버의 공인 IP |
| A | api | 백 서버의 공인 IP |
| CNAME | www.ncbt.site | versel 도메인 주소 |

- A레코드 : 도메인 -> IP 포워딩해준다.

- CNAME : 도메인 -> 도메인 포워딩해준다.

위 작업을 마친 뒤 DNS 서버에 포워딩 설정이 적용되도록 기다려야 한다. 이 단계는 생각보다 오래 걸린다 ! 우리는 24시간을 기다려야 했다...

``` bash
# nslookup 명령어로 포워딩 설정이 반영되었는지 확인할 수 있다.
nslookup api.ncbt.site

# 예시
root@s193e2809584:~# nslookup api.ncbt.site
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	api.ncbt.site
Address: 내가 설정한 공인IP # 이 주소가 내가 설정한 주소인지 확인 !
```

포워딩이 완료되었으면, 다시 4단계로 돌아가 인증서 발급을 마치면 된다.


> #### 5단계 : 인증서 파일 생성 확인

인증서 파일이 생성되었는지 확인하려면 아래 명령어로 해당 디렉토리 내의 파일 목록을 확인해보면 된다. fullchain.pem , privkey.pem 등 있어야 한다.

``` bash
ls -l /etc/letsencrypt/live/api.ncbt.site/

# 예시
total 4
lrwxrwxrwx 1 root root  37 Jan  3 14:03 cert.pem -> ../../archive/api.ncbt.site/cert1.pem
lrwxrwxrwx 1 root root  38 Jan  3 14:03 chain.pem -> ../../archive/api.ncbt.site/chain1.pem
lrwxrwxrwx 1 root root  42 Jan  3 14:03 fullchain.pem -> ../../archive/api.ncbt.site/fullchain1.pem
lrwxrwxrwx 1 root root  40 Jan  3 14:03 privkey.pem -> ../../archive/api.ncbt.site/privkey1.pem
-rw-r--r-- 1 root root 692 Jan  3 14:03 README
```

<br>

### 3️⃣ Apache SSL 설정

우리는 Apache 에 SSL 인증서를 적용하여 외부와 HTTPS 로 암호화된 통신을 하도록 할 것이다. 이 설정이 제대로 되어야만 외부에서 HTTPS 를 통해 Apache 로 요청을 보내고, Apache 가 이를 Spring 으로 제대로 전달할 수 있다.

<br>

> #### 1단계 : VirtualHost 설정

Apache 는 기본적으로 80(HTTP) 포트와 443(HTTPS) 포트를 분리해서 처리한다. SSL을 사용하는 서버는 443 포트에서 HTTPS 요청을 수신하므로, 이를 위한 SSL 전용 VirtualHost 설정이 필요하다.

``` bash
# 설정 파일 열기
vi /etc/apache2/sites-available/000-default.conf
```

80 포트에 대한 VirtualHost 설정만 있을 것이다. 여기에 443 포트에 대한 설정을 추가해줘야 한다. 

``` bash
<VirtualHost *:80>
    # 기존 HTTP 설정
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    # 로그 설정
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    # HTTP 요청을 HTTPS로 리디렉션
    Redirect permanent / https://api.ncbt.site/
</VirtualHost>

<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    ServerName api.ncbt.site

    # SSL 설정
    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/api.ncbt.site/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/api.ncbt.site/privkey.pem
    SSLCertificateChainFile /etc/letsencrypt/live/api.ncbt.site/chain.pem

    # 리버스 프록시 설정 (Spring 애플리케이션이 실행되는 8080 포트를 넣어준다)
    ProxyPass / http://localhost:8080/
    ProxyPassReverse / http://localhost:8080/

    # 로그 설정
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

<br>

> #### 2단계 : 리버스 프록시 설정

리버스 프록시란, Apache 가 https://api.ncbt.site 로 요청을 받으면, 이를 Docker 컨테이너에서 실행중인 Spring 애플리케이션으로 전달하도록 설정하는 것을 말한다. 위 ```<VirtualHost *:443> 블럭```에서 설정하면 되고, 유의할 점은 스프링이 실행되고 있는 IP 주소를 정확하게 입력해줘야 찾을 수 있다. 우리는 스프링 애플리케이션을 호스트 네트워크 모드로 실행시키고 있기 때문에 Apache 입장에서는 localhost 로 컨테이너에 접근하도록 해줘야 한다. docker-compose 를 사용하거나 브리지 네트워크 모드로 실행중이라면 컨테이너의 IP 주소를 확인하여 Apache 설정에 적용해야 한다.

<br>

> #### 3단계 : SSL 모듈 및 리버스 프록시 모듈 활성화

``` bash
# SSL 모듈 활성화
sudo a2enmod ssl

# 리버스 프록시 관련 모듈 활성화
sudo a2enmod proxy
sudo a2enmod proxy_http

# Apache 서버 재시작
sudo systemctl restart apache2
```

<br>

### 4️⃣ 스프링 컨테이너 실행

우리는 호스트 네트워크 모드로 실행했지만, docker-compose 혹은 브리지 네트워크로 실행해도 상관없다. 이 때는 리버스 프록시 설정에 유의하자 !

``` bash
# 컨테이너 실행
docker run -d \
--network host \
--name spring-container \
-e SPRING_DATASOURCE_URL=jdbc:mysql://내부IP:3306/ncbt \
-e SPRING_DATASOURCE_USERNAME=아이디 \
-e SPRING_DATASOURCE_PASSWORD=비밀번호 \
-e SPRING_DATASOURCE_DRIVER-CLASS-NAME=com.mysql.cj.jdbc.Driver \
ghkdusghd/ncbt:ver8
```

<br>

### 5️⃣ 실행 확인

위 과정까지 다 완료했다면, https://api.ncbt.site 도메인으로 요청하면 SSL 이 적용된 스프링 애플리케이션을 정상적으로 서비스할 수 있을 것이다. 

<br>

> #### 브라우저에서 확인

<br>

<img src="/assets/images/pages/projects/ncbt/스크린샷 2025-01-04 오후 12.50.46.png">

<br>

> #### 프론트 서버와 연결

이미 배포가 완료된 프론트 서버의 base_url 을 https://api.ncbt.site 로 수정하니 서버와 통신이 되는 것을 확인할 수 있었다! 그치만 당연스럽게도 개발단계에서 설정한 origin 설정을 변경하지 않았기에 CORS 오류가 발생했다.

<img src="/assets/images/pages/projects/ncbt/스크린샷 2025-01-04 오후 12.53.22.png">


<br>

아래 한 줄을 변경한 후 다시 컨테이너를 실행시켜보았지만 CORS 문제는 해결되지 않았다 ... 이건 다음 포스팅에서 잡아보도록 하자 !

``` java
config.addAllowedOrigin("https://www.ncbt.site");
```


<br>

🔖 참고문서

[Let's Encrypt 인증서 발급 및 갱신법](https://www.owl-dev.me/blog/42)