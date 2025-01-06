---
title: "[CORS] Apache CORS 에러 해결"
parent: NCBT
nav_order: -11

tags:
  - [apache, proxy]

toc: true
toc_sticky: true

date: 2025-01-04
last_modified_at: 2025-01-04
---

> 이전 포스팅에서는 프론트 서버와 백 서버가 HTTPS 로 통신하도록 Apache 서버에 SSL 인증서를 설치하고, 리버스 프록시 설정으로 스프링 부트 애플리케이션에 요청이 전달되도록 만들었다. 이로 인해 혼합 콘텐츠 오류는 해결했지만, CORS 오류가 새롭게 발생했다.

### 👩🏻‍💻 에러 원인 찾기

> #### 1단계 : CORS 정책 변경

제일 처음으로 스프링 부트 CORS 정책에서 addAllowedOrigin 설정을 실제 프론트 서버의 도메인으로 변경했다. 그러나 여전히 CORS 오류가 발생했는데... 뭔가 이상했다.

``` java
config.addAllowedOrigin("https://www.ncbt.site");
```

<img src="/assets/images/pages/projects/ncbt/스크린샷 2025-01-04 오후 12.53.22.png">

preflight request 라는 용어가 신경쓰여서 찾아보니, CORS 가 동작하는 방식은 요청에 따라 달라진다고 한다.

| 종류 | 메서드 | Content-Type |
|---|---|---|
| Simple Request | GET, POST, HEAD | text/plain, application/x-www-form-urlencoded, multipart/form-data |
| Preflight Request | Simple Request 가 아닌 경우가 해당한다 |

단순 요청이 아닌 경우, 브라우저는 요청을 한 번에 보내지 않고 사전요청과 본 요청으로 나누어 서버에 전송한다고 하고 이 때의 요청이 Preflight Request 가 되는 것이다. 이 경우 OPTIONS 메서드를 사용한다고 한다. 

나는 GET 요청을 보냈지만 Content-Type 을 application/json 으로 요청했기에 Preflight Request 로 요청된 듯 했다. 그래서 아래 설정을 추가했지만 여전히 해결되지 않았다.

``` java
// OPTIONS 메서드 추가
config.addAllowedMethod("OPTIONS");
```

<br>

> #### 2단계 : 에러 로그 확인

여전히 감이 오지 않아서 스프링 에러 로그를 확인해 보았더니 HTTP request header 를 파싱하는 과정에서 오류가 발생한 것 같았다.

``` bash
2025-01-04T04:20:41.214Z  INFO 1 --- [backend] [io-8080-exec-10] o.apache.coyote.http11.Http11Processor   : Error parsing HTTP request header
 Note: further occurrences of HTTP request parsing errors will be logged at DEBUG level.

java.lang.IllegalArgumentException: Invalid character found in method name [0x050x010x00...]. HTTP method names must be tokens
	at org.apache.coyote.http11.Http11InputBuffer.parseRequestLine(Http11InputBuffer.java:407) ~[tomcat-embed-core-10.1.28.jar!/:na]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:257) ~[tomcat-embed-core-10.1.28.jar!/:na]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:63) ~[tomcat-embed-core-10.1.28.jar!/:na]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:904) ~[tomcat-embed-core-10.1.28.jar!/:na]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1741) ~[tomcat-embed-core-10.1.28.jar!/:na]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:52) ~[tomcat-embed-core-10.1.28.jar!/:na]
	at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1190) ~[tomcat-embed-core-10.1.28.jar!/:na]
	at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659) ~[tomcat-embed-core-10.1.28.jar!/:na]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:63) ~[tomcat-embed-core-10.1.28.jar!/:na]
	at java.base/java.lang.Thread.run(Thread.java:833) ~[na:na]
```

찾아보니 리버스 프록시로 요청을 전달하는 과정에서 요청을 수정하거나 손상시킬 수 있다고 하여 아파치 서버의 에러 로그도 확인해보았다.

``` bash
# Apache 서버로 들어오는 모든 요청 확인
tail -f /var/log/apache2/access.log

# 출력된 로그
122.35.170.152 - - [04/Jan/2025:13:08:48 +0900] "OPTIONS /refreshToken HTTP/1.1" 403 4180 "https://www.ncbt.site/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
18.191.32.67 - - [04/Jan/2025:13:11:47 +0900] "\x16\x03\x01" 400 483 "-" "-"
122.35.170.152 - - [04/Jan/2025:13:12:28 +0900] "OPTIONS /refreshToken HTTP/1.1" 403 1021 "https://www.ncbt.site/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
122.35.170.152 - - [04/Jan/2025:13:12:40 +0900] "OPTIONS /refreshToken HTTP/1.1" 403 1021 "https://www.ncbt.site/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
18.191.32.67 - - [04/Jan/2025:13:12:47 +0900] "GET / HTTP/1.1" 301 513 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) Chrome/126.0.0.0 Safari/537.36"
122.35.170.152 - - [04/Jan/2025:13:12:50 +0900] "OPTIONS /refreshToken HTTP/1.1" 403 1021 "https://www.ncbt.site/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
122.35.170.152 - - [04/Jan/2025:13:12:50 +0900] "OPTIONS /ranking/v2 HTTP/1.1" 403 489 "https://www.ncbt.site/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
122.35.170.152 - - [04/Jan/2025:13:22:43 +0900] "OPTIONS /refreshToken HTTP/1.1" 403 4180 "https://www.ncbt.site/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
122.35.170.152 - - [04/Jan/2025:13:22:52 +0900] "GET /form/email-code?email=hwy9597%40naver.com HTTP/1.1" 403 999 "https://www.ncbt.site/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
```

내가 보낸 모든 요청이 Apache 서버에서 403 에러로 반환되는 것을 확인했다. 생각해보면 클라이언트의 요청이 일단 Apache 서버로 들어가고, Apache 가 스프링으로 리버스 프록시 해주는 것이기 때문에... Apache 에서도 CORS 헤더를 추가해야겠다 싶었다.

<br>

### 🚀 해결

> #### 1단계 : Apache 에서 CORS 설정

``` bash
# VirtualHost 설정파일 열기
vi /etc/apache2/sites-available/000-default.conf
```

``` bash
# VirtualHost :*443 블럭에 아래 코드 추가

<VirtualHost *:443>
ServerAdmin yeonguo95@gmail.com # 서버 에러시 사용자에게 안내될 이메일
DocumentRoot /var/www/html # 이건 그대로 냅두기
ServerName api.ncbt.site # 서버 도메인

# 리버스 프록시 관련 설정
ProxyRequests Off
ProxyPreserveHost On

# Apache 에 디렉토리 권한 부여
<Directory /var/www/html>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
<Directory /var/www>
    Require all granted
</Directory>

# CORS 헤더 추가
<IfModule mod_headers.c>
Header always set Access-Control-Allow-Origin "https://www.ncbt.site"
Header always set Access-Control-Allow-Methods "OPTIONS, GET, POST, PUT, DELETE"
Header always set Access-Control-Allow-Headers "Authorization, Content-Type, X-Custom-Header, Set-Cookie"
Header always set Access-Control-Allow-Credentials "true"
Header always set Access-Control-Expose-Headers "Authorization, Set-Cookie"
</IfModule>

# 리버스 프록시 활성화
<IfModule mod_proxy.c>
    ProxyPreserveHost On
    ProxyPass / http://localhost:8080/
    ProxyPassReverse / http://localhost:8080/
</IfModule>

# 프리플라이트 OPTIONS 요청 처리
RewriteEngine On
RewriteCond %{REQUEST_METHOD} OPTIONS
RewriteRule ^(.*)$ $1 [R=200,L]
</VirtualHost>
```

``` bash
# 관련 모듈 활성화
sudo a2enmod headers
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_http

# 아파치 재시작
sudo systemctl restart apache2
```

이후 다시 시도해보니, 오류 메시지가 조금 변했다. Access-Control-Allow-Origin 헤더에 다중값이 들어갔다는 것...

<img src="/assets/images/pages/projects/ncbt/스크린샷 2025-01-04 오후 3.13.59.png">

<br>

> #### 2단계 : Spring CORS 설정 삭제

왜 헤더에 다중값이 들어갈까? 생각해보니 우리는 Spring 서버와 바로 통신하는 것이 아닌, 로드밸런서 역할의 Apache 서버와 통신하고 있다. 그러므로 CORS 설정은 Apache 에'만' 해주고, Spring 쪽의 CORS 설정은 삭제해야 하는 것이다. 그래서 관련 설정을 전부 주석 처리했다.

<br>

> #### 3단계 : Docker 실행 매커니즘 변경

계속 진행했음에도 CORS 에러가 계속 발생했다. 그래서 번뜩 든 생각이... 도커 허브의 문제가 아닐까? 싶었다. 왜냐하면 요청이 들어간다면 분명 스프링 컨테이너에 로그가 뜰텐데 아무것도 뜨지 않는 것이다. 혹시라도 예전 버전의 코드가 갱신되지 않고 있는 게 아닌가 싶었다. (특히 도커 허브에서는 코드를 살펴볼 수가 없어서 변경사항이 반영되었는지 알기 어렵다) 

그래서 결국 Github 를 사용해서 컨테이너를 실행시키기로 했다.

``` bash
# 3-1. (로컬PC에서) 소스코드를 깃허브에 푸시
git push origin main 

# 3-2. (클라우드에서) 소스코드 내려받기
git pull origin main

# 3-3. 빌드 실행
./gradlew clean build -x test

# 3-4. 도커 이미지 생성
docker build -t 도커이미지명:버전 .

# 3-5. 도커 컨테이너 실행
docker run -d \
--network host \
--name spring-container \
-e SPRING_DATASOURCE_URL=jdbc:mysql://내부IP:3306/ncbt \
-e SPRING_DATASOURCE_USERNAME=아이디 \
-e SPRING_DATASOURCE_PASSWORD=비밀번호 \
-e SPRING_DATASOURCE_DRIVER_CLASS_NAME=com.mysql.cj.jdbc.Driver \
도커이미지명:버전
```

이렇게 했더니 결국... !! CORS 에러가 사라지고 정상적으로 통신이 가능해졌다 !! 고생 끝에 배포 성공 만만세다 🙌🏻

<img src="/assets/images/pages/projects/ncbt/스크린샷 2025-01-06 오후 4.04.57.png">

<br>

### 🤓 배운점

> CORS 동작 원리

``` markdown
배포 환경에서 신경써야 할 점이 매우 많았는데, CORS 는 정말 된통 고생했다.
로컬에서 개발할 때는 그냥 리액트 포트 localhost:3000 만 허용해주면 되니까 문제가 없었는데,
배포를 하고 SSL 인증서를 적용해보니 고려해야 할 점이 많았다.
```

#### preflight request

CORS 의 simple request 와 달리, 먼저 OPTIONS 메서드를 통해 실제 요청을 전송하기에 안전한지 확인한다. 200번대 응답을 받으면 다시 원래의 요청을 보낸다.

#### 왜 POSTMAN 에서는 CORS 오류가 발생하지 않을까 ?

CORS 정책 위반 판단은 '브라우저' 가 실행한다. 따라서 브라우저 없이 서버 간 통신을 진행한다면 당연히 CORS 에러를 확인할 수 없는 것이다.

> Apache 서버에 대한 이해

``` markdown
애초에 Apache 를 설치한 이유는 SSL 인증서가 웹 서버에서 동작하기 때문이었다.
그렇다 보니 Apache 서버는 단순히 SSL 설정을 위한 것이라고 생각이 굳어지게 되었는데,
실상은 Load Balancer 의 역할도 겸한다.
외부에서 https://api.ncbt.site 로 요청이 들어오는 경우 443 포트에서 동작하는 Apache 가 먼저 요청을 받는다.
그리고 리버스 프록시로 해당 도메인에 대한 요청을 스프링 애플리케이션으로 보내주는 것이다.
말하자면 대문 역할인 것인데, 이걸 깊이 생각하지 못하고 하던대로 스프링에서 CORS 설정을 하려고 하니 문제가 생긴 것이었다.
Apache 와 스프링은 localhost 로 내부적인 통신을 하고 있으니 서로 간의 CORS 설정은 필요가 없다.
따라서, 대문이 되는 Apache 에서만 CORS 설정을 해주면 되는 것이었다.
```


<br>

🔖 참고자료

[Preflight Request 란?](https://velog.io/@gygy/ExpressNode.js-CORS-%EC%9D%B4%EC%8A%88-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0)