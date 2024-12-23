---
title: "[NaverCloud + DockerHub] 네이버 클라우드 + 도커로 스프링 부트 프로젝트 배포하기"
parent: NCBT
nav_order: 8

tags:
  - [NCP, docker]

toc: true
toc_sticky: true

date: 2024-12-17
last_modified_at: 2024-12-17
---

>드디어 약 2달간 준비해온 NCBT 서비스를 배포한다. 이전에 학원에서 진행했던 프로젝트 때는 같은 팀 전공자 친구가 혼자 디비, 서버, 프론트 전부를 배포해주어서 내가 배울 수 있는 기회가 없었기에 이번 프로젝트의 대(大)목표는 '나도 배포를 해보자!' 는 것이었다. 상상 이상으로 만만치 않았기에... 미래의 나, 너, 우리를 위해 어떤 과정으로 배포가 이루어지는지 그 과정을 자세히 정리해보자.

> ### 1️⃣  클라우드 생성

'배포한다' 라는 것은 웹사이트를 실제 사용자가 접근할 수 있는 환경에 공개하는 것을 의미한다. 우리는 지금까지 열심히 만든 코드를 웹 서버 (네이버 클라우드) 에 올려서 다른 사람들도 접근할 수 있게 할 것이다.

#### 네이버 클라우드

``` markdown
'네이버 클라우드' 를 선택한 이유는 우리가 네이버 클라우드 캠프 수료생이라,
당시 약 200만원의 바우처를 받아 이래저래 연습을 해봤기 때문에 익숙했기 때문에 선정했다.
```

#### 서버 스펙 - High-CPU server (vCPU 2EA, Memory 4GB)

``` markdown
처음에는 비용을 고려하여... micro server 를 선택했다.
다만, 메모리 용량이 적어서 그런지 도커 실행이 되지 않았다 ㅠㅠ
💡 참고로, micro server 는 네이버 클라우드 가입일 직후 1년동안 무료로 제공된다.
그래서 서버 스펙을 조금 더 높이기로 했고, 그나마 6만원대의 서버를 생성했다..
```

> ### 2️⃣  도커 설치


#### 1단계 : 도커 설치에 필요한 의존성 패키지 설치

``` bash
# 우분투 시스템 패키지 업데이트
sudo apt-get update

# 도커 설치에 필요한 의존성 패키지 설치
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
```

#### 2단계 : 도커 공식 GPG 키 추가

``` bash
# 도커의 공식 GPG 키 추가
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

#### 3단계 : 도커 저장소 추가

``` bash
# 도커 저장소 추가
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### 4단계 : 도커 최신버전 설치

``` bash
# 설치 전 한번 더 업데이트..
sudo apt-get update

# 도커 설치
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

#### 5단계 : 도커 서비스 시작 및 확인

``` bash
# 도커 서비스 시작
sudo systemctl start docker

# 도커가 정상적으로 실행되고 있는지 확인
# ACTIVE 상태인 것 확인하고 ctrl + c 로 빠져나오기
sudo systemctl status docker

# 도커 버전 확인
docker --version
```

> ### 3️⃣  MySQL DB 설치

도커 이미지를 생성하기 전에, 먼저 클론해온 애플리케이션을 빌드해야한다. 나는 gradle 로 빌드했는데, 클라우드 서버에 MySQL 이 설치되어 있지 않기 때문에 DB 연결 실패로 빌드가 정상적으로 실행되지 않았다.


그렇다면 선택지는 < 1. 네이버 클라우드 서버에 MySQL 을 설치한다 / 2. MySQL 도 별도로 배포한다 > 두 가지일 것이다. 처음에는 2번 방식으로 하려고 했으나, 우리가 소규모 프로젝트라 테이블도 그리 많지 않고, 용량도 많이 차지하지 않을 것이라 예상하여 (많아봐야 1GB 정도로 예상됨) 결국 1번 방식을 선택했다.

혹시라도 디스크 용량이 부족하여 증량해야 할 경우를 감안하여 비용 계산기를 두드려보았다. 현재 디스크 용량이 10GB 이고, OS 및 도커 등 서버 운영에 꼭 필요한 파일들의 용량이 약 6.5GB 를 차지하고 있으니, 증량해봐야 10GB 정도가 될 것이다.

<p style="color:gray">V1. 기존 비용</p>
<img width="667" alt="스크린샷 2024-12-23 오후 12 18 53" src="https://github.com/user-attachments/assets/da081a86-7a2d-422c-a007-52960f1c87af" />


<p style="color:gray">V2. 10GB 증량 후 비용</p>
<img width="662" alt="스크린샷 2024-12-23 오후 12 18 30" src="https://github.com/user-attachments/assets/adb8c9e3-0319-488d-a0eb-1c3404b13a7c" />

약 1,100원의 차이라 이 정도면 충분히 감당 가능한 비용이기도 하고, 이렇게 하면 운영상의 편의성이 있을 것이고 타 서버와의 통신으로 인한 보안 위험까지 없어질것이라 생각하여 1번 방안을 선택하기로 했다.

#### 1단계 : 패키지 업데이트

``` bash
sudo apt update
```

#### 2단계 : MySQL 서버 설치

``` bash
sudo apt install mysql-server
```

#### 3단계 : MySQL 서비스 시작 및 활성화

``` bash
# MySQL 서비스 상태 확인
# Active 인것 확인 후 ctrl + c 로 빠져나오기
sudo systemctl status mysql

# MySQL 서비스가 부팅 시 자동으로 시작되도록 설정
sudo systemctl enable mysql
```

#### 4단계 : MySQL 접속 및 비밀번호 설정

``` bash
# MySQL 접속 (초기 비밀번호는 없으므로 그냥 엔터키를 치면 들어가진다)
sudo mysql -u root -p

# 비밀번호 설정
ALTER USER 'root'@'%' IDENTIFIED BY '새로운_비밀번호';

# 변경사항 적용
FLUSH PRIVILEGES;

# 로그아웃 후 다시 로그인해봅시다.
exit
```

- 중요한 점은, root 계정 혹은 사용자 계정의 권한을 '%' 로 주어야 외부에서 접속할 수 있다는 점이다.

#### 5단계 : 데이터베이스 생성 및 테이블 생성

``` bash
# 데이터베이스 생성 및 선택
CREATE DATABASE ncbt;
USE ncbt;

# 테이블 생성 후 확인
SHOW tables;
```

#### 6단계 : bind-address 설정

``` bash
# MySQL 설정 파일
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

# 위 파일을 열고, 아래 한 줄을 추가한 후 저장한다.
bind-address            = 0.0.0.0

ctrl + O # 저장

Enter # 파일명 확인

ctrl + X # 나가기
```

여기까지 진행했을 때 약 0.4GB 추가되는 것을 확인했다. 운영에 무리가 가지 않을 만한 용량이라 괜찮다고 판단 후 다음으로 진행하기로 했다.

> ### 4️⃣  Docker Hub 에 스프링 이미지 푸시

- 나는 깃허브가 아닌 Docker Hub 를 사용하여 배포할 것이기 때문에, 프로젝트를 개발한 로컬 PC 에서 도커 이미지를 만든 후 Docker Hub 로 푸시할 것이다. (Docker Hub 는 깃허브와 비슷하게 사용할 수 있는 도커의 버전관리 도구이다)

#### 1단계 : Dockerfile 생성

``` bash
FROM openjdk:17-slim

USER root
RUN apt-get update && apt-get install -y default-mysql-client

ARG JAR_FILE=build/libs/backend-0.0.1-SNAPSHOT.jar

COPY ${JAR_FILE} app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

#### 2단계 : 로컬에 도커 설치

``` bash
# 로컬 PC 에서 2️⃣ 작업을 수행하면 된다.
```

#### 3단계 : 빌드

``` bash
# 프로젝트 루트 디렉토리로 이동한 후 아래 명령어로 실행시킨다.
./gradlew build -x test

# 만약 ./gradlew 명령어로 실행시킬 수 없다면 아래 순서대로 진행.
sdk install gradle

gradle wrapper

chmod +x gradlew
```

#### 4단계 : 도커 로그인

``` bash
# 아래 명령어를 입력하면 터미널 창에 링크가 하나 뜰 것이다.
# 그 링크로 접속하여 로그인하면 된다.
docker login
```

#### 5단계 : 도커 허브에 저장소 생성

<img width="1429" alt="스크린샷 2024-12-23 오후 12 29 47" src="https://github.com/user-attachments/assets/dd6978e0-bb94-4e3a-a68f-5ff7b8df3760" />

- [Docker Hub (클릭하면 이동)](https://hub.docker.com/) 에 로그인한 이후, 상단에 Repositories 를 클릭

- Create Repository 버튼을 눌러 저장소를 생성한다. public 으로 생성해야 한다.

#### 6단계 : 도커 허브로 스프링 애플리케이션 이미지 푸시

``` bash
# 도커 이미지를 생성하는 동시에 허브로 푸시하는 명령어
docker buildx build --platform linux/amd64 -t 도커허브계정닉네임/저장소명:태그 --push .

# ex
docker buildx build --platform linux/amd64 -t ghkdusghd/ncbt:ver1 --push .
```

#### 7단계 : 이미지가 허브에 잘 올라갔는지 확인

<img width="677" alt="스크린샷 2024-12-23 오후 12 33 38" src="https://github.com/user-attachments/assets/f9df8030-db45-4017-8ac1-41e7d3eadcdc" />

> ### 5️⃣ 클라우드 서버에서 도커 이미지 받아오기

``` bash
# 도커 허브에 저장된 이미지들 중 원하는 버전을 가져오면 된다.
docker pull ghkdusghd/ncbt:ver8
```

#### 도커 컨테이너 실행

``` bash
# 컨테이너 실행
docker run -d \
--name spring-container \
--network host \
-e SPRING_DATASOURCE_URL=jdbc:mysql://서버내부IP:3306/ncbt \
-e SPRING_DATASOURCE_USERNAME=아이디 \
-e SPRING_DATASOURCE_PASSWORD=비밀번호 \
-e SPRING_DATASOURCE_DRIVER-CLASS-NAME=com.mysql.cj.jdbc.Driver \
ghkdusghd/ncbt:ver8
```
- network : 나의 경우 MySQL 서버를 클라우드 서버의 로컬 환경에 설치했기 때문에 (컨테이너화 하지 않음) host 로 설정해줬다. 참고로, 컨테이너가 통신하려면 통신하려는 대상 서버와 같은 네트워크에 있어야 한다.

- datasource url : MySQL 은 로컬 환경에 설치했기 때문에, 클라우드 서버의 내부 IP 로 접근해야 한다. 

  ``` bash
  # 서버의 내부 IP 확인 (이더넷 eth0 주소를 확인하면 된다.)
  ip addr show
  ```

#### 🤯 실행 실패 !

여기까지 잘 왔다고 생각했는데 스프링 컨테이너가 실행되지 않고 종료되는 문제가 발생했다. 스프링 컨테이너의 로그를 확인해보니 ```Failed to obtain JDBC Connection``` 라는 오류가 발생하고 있었다.

> ### 6️⃣  MySQL 도메인 네임과 내부 IP 매핑

로컬에서 실행중인 MySQL 에러로그를 확인해보았다.

``` bash
# 로그 확인하는 명령어
sudo tail -f /var/log/mysql/error.log
```

캡쳐는 못했지만... MySQL 의 도메인이 '@!!@%(예시)' 인데, 스프링 컨테이너가 접근을 시도하는 내부 IP 는 '10.0.0.7(예시)' 라서 찾을 수 없다는 로그를 발견했다.

#### 도메인 매핑

``` bash
# /etc/hosts 파일 수정
sudo nano /etc/hosts

# 도메인 이름과 내부 IP 를 매핑하는 항목을 추가한다
192.168.1.100    mysql.example.com

ctrl + O # 저장

Enter # 파일명 확인

ctrl + X # 종료
```

이렇게 한 뒤 다시 docker run 으로 실행하면 스프링 애플리케이션이 문제없이 실행될 것이다.

> ### 7️⃣ POSTMAN 테스트

이제 서버를 배포했으니 ```공인IP:8080``` 포트로 스프링 애플리케이션에 접근할 수 있을 것이다. 아직 프론트 서버를 배포하기 전이니 포스트맨으로 테스트 해보자 ! (테스트 전에 클라우드 서버의 ACG 인바운드 설정에 8080 포트를 열어줘야 한다)

<img width="534" alt="스크린샷 2024-12-23 오후 12 59 34" src="https://github.com/user-attachments/assets/f7612085-d831-49d0-88c8-e3b371802f64" />

403 에러로 실패하긴 했지만, 서버와 통신이 되긴 했다. 다음 포스팅에서 프론트 서버를 배포한 이후 에러를 잡아나가 보기로 하겠다.

---

> ### ✏️ 왜 젠킨스를 사용하지 않았는가?

    1. 서버의 디스크 용량이 작다.
    젠킨스를 사용하면 빌드가 쌓이면서 서버에서 차지하는 디스크 용량도 쌓이게 되는데, 젠킨스에 미숙하다보니 빌드 테스트를 계속하면서 점차 쌓이는 용량을 관리하기 버겁게 되었다.

    2. 배포 초기단계를 제외하면 코드 수정이 잦지 않을 것이다.
    우리 서비스의 주요 데이터라 할 수 있는 '문제' 데이터는 통신비용 및 시간 절감을 위해 프론트 서버에 저장하기로 했다. 때문에 백엔드 서버에서는 문제 데이터를 건드릴 일이 없었고, 프로젝트 규모도 크지 않기 때문에 그렇게 자주 수정할까? 싶어 일일이 배포하는 방법을 선택하기로 했다.

> ### 🤓 배운 점

#### 트러블슈팅

    네트워크에 대한 이해가 부족하여, 컨테이너가 통신할 대상의 내부 IP 만 입력해주면 문제가 없을 것이라 생각했다. 그런데 왜 JDBC 연결 오류가 발생했을까?

  - 스프링 애플리케이션이 컨테이너 내부에서 실행되고 있고, 그 컨테이너가 호스트 머신의 MySQL 서버에 접근하려고 할 때, 만약 컨테이너 내부에서 localhost 혹은 127.0.0.1 을 사용하면 자기 자신을 참조하는 문제가 발생해 버린다. 때문에 컨테이너는 도메인 이름을 사용해서 MySQL 과 연결을 시도하려 한 듯 하다.

  - 나는 MySQL 의 도메인을 별도로 지정한 적이 없었지만, 아마 상기 이유로 발생한 문제가 아닐까 싶다...