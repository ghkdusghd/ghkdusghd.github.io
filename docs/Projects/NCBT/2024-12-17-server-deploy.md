---
title: "[NaverCloud + Docker] 네이버 클라우드 + 도커로 스프링 부트 서버 배포하기"
parent: NCBT
nav_order: 8

tags:
  - [NCP, jenkins, docker]

toc: true
toc_sticky: true

date: 2024-12-17
last_modified_at: 2024-12-17
---

>드디어 약 2달간 준비해온 NCBT 서비스를 배포한다. 이전에 학원에서 진행했던 프로젝트 때는 같은 팀 전공자 친구가 혼자 디비, 서버, 프론트 전부를 배포해주어서 내가 배울 수 있는 기회가 없었기에 이번 프로젝트의 대(大)목표는 '나도 배포를 해보자!' 는 것이었다. 상상 이상으로 만만치 않았기에... 미래의 나, 너, 우리를 위해 어떤 과정으로 배포가 이루어지는지 그 과정을 자세히 정리해보자.

### 1️⃣  클라우드 생성

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

### 2️⃣  도커 설치


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

### 3️⃣  깃허브 저장소 클론


#### 1단계 : 프로젝트 디렉토리 생성 후 이동

``` bash
# 프로젝트를 클론할 로컬 저장소를 만들자
sudo mkdir -p /docker_projects/프로젝트명/project

# 이동
cd /docker_projects/프로젝트명/project
```

#### 2단계 : Dockerfile 생성

``` bash
FROM openjdk:17-slim

USER root
RUN apt-get update && apt-get install -y default-mysql-client

ARG JAR_FILE=build/libs/backend-0.0.1-SNAPSHOT.jar

COPY ${JAR_FILE} app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]

```

클론할 저장소 안에 Dockerfile 이 있어야 한다. 도커 파일이란, 도커 이미지를 만들기 위한 청사진이라고 생각하면 되고, 나의 경우는 이전 프로젝트 때 동료들이 만들어둔 파일을 참고해서 만들었다. Dockerfile 을 깃허브에 푸쉬해놓고 다음 단계로 넘어가자.


#### 3단계 : 깃 리포지토리 클론

``` bash
git clone <저장소 URL>
```

### 4️⃣  MySQL DB 설치

도커 이미지를 생성하기 전에, 먼저 클론해온 애플리케이션을 빌드해야한다. 나는 gradle 로 빌드했는데, 클라우드 서버에 MySQL 이 설치되어 있지 않기 때문에 DB 연결 실패로 빌드가 정상적으로 실행되지 않았다.


그렇다면 선택지는 < 1. 네이버 클라우드 서버에 MySQL 을 설치한다 / 2. MySQL 도 별도로 배포한다 > 두 가지일 것이다. 처음에는 2번 방식으로 하려고 했으나, 우리가 소규모 프로젝트라 테이블도 그리 많지 않고, 용량도 많이 차지하지 않을 것이라 예상하여 (많아봐야 1GB 정도로 예상됨) 결국 1번 방식을 선택했다.

혹시라도 디스크 용량이 부족하여 증량해야 할 경우를 감안하여 비용 계산기를 두드려보았다. 현재 디스크 용량이 10GB 이고, OS 및 도커 등 서버 운영에 꼭 필요한 파일들의 용량이 약 6.5GB 를 차지하고 있으니, 증량해봐야 10GB 정도가 될 것이다.

<p style="color:gray">V1. 기존 비용</p>
<img width="679" alt="스크린샷 2024-12-18 오후 2 05 15" src="https://github.com/user-attachments/assets/e8c735c7-ba7e-4d4c-a420-ce3a0895e974" />

<p style="color:gray">V2. 10GB 증량 후 비용</p>
<img width="677" alt="스크린샷 2024-12-18 오후 2 05 41" src="https://github.com/user-attachments/assets/5bb56d34-64e1-4580-a604-de1498b84e24" />

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
ALTER USER 'root'@'localhost' IDENTIFIED BY '새로운_비밀번호';

# 변경사항 적용
FLUSH PRIVILEGES;

# 로그아웃 후 다시 로그인해봅시다.
exit
```

#### 5단계 : 데이터베이스 생성 및 테이블 생성

``` bash
# 데이터베이스 생성 및 선택
CREATE DATABASE ncbt;
USE ncbt;

# 테이블 생성 후 확인
SHOW tables;
```

여기까지 진행했을 때 약 0.4GB 추가되는 것을 확인했다. 운영에 무리가 가지 않을 만한 용량이라 괜찮다고 판단 후 다음으로 진행하기로 했다.

### 5️⃣  환경변수 설정


#### 1단계 : application.yml 생성

``` bash
# application.yml 은 java main/resources 아래 경로에 있어야 한다.
# 해당 경로로 이동 후 아래 명령어 실행.. (nano 는 파일 생성 후 편집기를 열게 하는 옵션이다.)
sudo nano application.yml

# 저장
ctrl + O  

# 파일명 확인
Enter

# 편집기 종료
ctrl + X
```

깃허브에는 보안상 문제로 환경변수를 push 하지 않으니, 당연히 깃허브에서 pull 받은 파일에는 환경변수가 없다. 그러니 서버에도 환경변수를 적어줘야 한다.

#### 2단계 : 빌드 (혹은 재빌드)

``` bash
# Gradle로 빌드하기 (build/libs 경로에 jar 파일이 생성된다)
./gradlew build

# 혹시 /build/libs 경로에 jar 파일이 이미 있다면 삭제 후 재빌드 해보자.
rm -f build/libs/파일명.jar
./gradlew clean #빌드 클린
```

#### 🤯 빌드 실패 !

- 테스트 단계에서 DB CONNECTION 에러가 발생했다... 그렇지만 일단 스킵하고 진행하기로 했다...

``` bash
# 테스트 단계를 스킵할 수 있는 명령어
./gradlew build -x test
```


### 6️⃣  MySQL 컨테이너 실행


#### 1단계 : MySQL 도커 이미지 생성

``` bash
# Docker hub 에서 MySQL 이미지 가져오기
docker pull mysql:latest

# 도커 이미지가 생성되었는지 확인 -> mysql 이라는 이미지가 있으면 성공
docker images
```

#### 2단계 : 컨테이너 실행

``` bash
docker run --name mysql-container -e MYSQL_ROOT_PASSWORD=비밀번호 -e MYSQL_DATABASE=데이터베이스명 -p 3306:3306 -d mysql:latest
```

#### 🤯 실행이 안된다 ?!

- MySQL 컨테이너가 사용하려는 3306 포트가 이미 로컬에서 사용중이기 때문에 그렇다.

``` bash
# 3306 포트를 사용하는 프로세스 표시
sudo lsof -i :3306

# 로컬 MySQL 이 실행중이라면, 아래 명령어로 정지하고 다시 docker run 실행하자.
sudo systemctl stop mysql
```

### 7️⃣  스프링 부트 컨테이너 실행

#### 1단계 : 스프링 도커 이미지 생성

``` bash
docker build -t <프로젝트명(IDE)> .
```

#### 2단계 : 컨테이너 실행

``` bash
# 컨테이너 실행
docker run -d -p 8080:8080 --name spring-container <도커이미지명>

# 실행 확인
docker ps

# 목록에 없다면 종료된 것이다.
# 아래 명령어는 실행중이지 않은 컨테이너도 보여준다.
docker ps -a

# -a 옵션에서 스프링 컨테이너가 없다면 1단계부터 다시 해보고, 
# 컨테이너가 있다면 에러 로그를 확인해보자.
docker logs <컨테이너 이름 또는 ID>
```

### 8️⃣  Docker 네트워크 설정

여기까지 잘 왔다고 생각했는데 스프링 컨테이너가 실행되지 않고 종료되는 문제가 발생했다. 로그를 확인해보니 ```Failed to obtain JDBC Connection``` 라는 오류가 잡히는데... 분명 같은 서버에 생성한 두 개의 컨테이너인데 어째서 연결 오류가 뜨는 것일까? 

- 처음에는 환경변수가 제대로 들어가지 않은 것이라고 생각해 .env 파일, application.yml 파일을 점검했으나 문제가 없었고, docker run 명령어를 실행할 때 -e 옵션으로 직접 환경변수를 넣어줬지만... 그래도 여전히 연결이 되지 않았다.

- 그렇다면? 도커 컨테이너는 같은 OS 위에 있어도 네트워크 설정을 따로 해주어야 할까? 라는 생각이 들어 찾아보았다.

``` markdown
기본적으로 MySQL 컨테이너와 Spring 애플리케이션 컨테이너는 동일한 네트워크에 있어야 통신이 가능하다고 한다.
네트워크에 연결하는 방법은 docker run 으로 컨테이너를 실행할 때 --network 옵션으로 설정할 수 있다.
이렇게 MySQL 과 Spring 애플리케이션이 동일한 네트워크에 있을 경우, localhost 대신 컨테이너 이름을 사용할 수 있다.
환경변수에서 DB URL 을 작성할 때 아래와 같이 작성하면 된다.
spring.datasource.url=jdbc:mysql://컨테이너명:3306/데이터베이스명
```

#### 1단계 : 네트워크 생성

``` bash
# 현재 생성된 네트워크를 확인하는 명령어
# 이 안에서 선택해도 되지만, 커스텀 네트워크를 따로 만들 수 있다.
docker network ls

# 커스텀 네트워크 생성 명령어
docker network create 네트워크명
```

#### 2단계 : MySQL 컨테이너를 커스텀 네트워크에 연결

``` bash
# 컨테이너 실행할 때 네트워크에 연결할 수 있으며,
docker run --network 네트워크명 \
--name mysql-container \
-e MYSQL_ROOT_PASSWORD=비밀번호 \
-e MYSQL_DATABASE=데이터베이스명 \
-p 3306:3306 -d mysql:latest

# 이미 실행한 컨테이너라면 아래 명령어로 네트워크에 연결해줄 수 있다.
docker network connect 네트워크명 컨테이너명
```

#### 3단계 : 스프링 컨테이너를 커스텀 네트워크에 연결

``` bash
# 컨테이너 실행할 때 네트워크에 연결
docker run --network 네트워크명 \
-d -p 8080:8080 \
--name spring-container \
<도커이미지명>

# 이미 실행중인 컨테이너를 네트워크에 연결
docker network connect 네트워크명 컨테이너명
```

#### 4단계 : 네트워크 연결 확인

``` bash
docker inspect 컨테이너명 | grep -i "Network"
```

두 개의 컨테이너가 같은 네트워크에 있는지 확인하자. 나의 경우 MySQL 컨테이너에 연결된 네트워크가 두 개였기에, 스프링 컨테이너와 동일한 네트워크는 남겨두고, 필요없는 네트워크는 지우기로 했다.

#### 5단계 : 네트워크 재설정

``` bash
# 네트워크 연결을 재설정하려면, 컨테이너를 삭제하고 재시작하면서 네트워크를 설정해야 한다.

# 모든 컨테이너 확인
docker ps -a

# 컨테이너 정지
docker stop 컨테이너명

# 컨테이너 삭제
docker rm 컨테이너ID

# 컨테이너를 재시작하면서 네트워크 설정
docker run --network 네트워크명 \
--name mysql-container \
-e MYSQL_ROOT_PASSWORD=비밀번호 \
-e MYSQL_DATABASE=데이터베이스명 \
-p 3306:3306 -d mysql:latest

# 네트워크 연결 확인
docker inspect 컨테이너명 | grep -i "Network"
```

이후 스프링 컨테이너를 실행했는데 여전히 JDBC 연결이 되지 않았다. 어라 어째서지...? 혼란스러웠으나, MySQL 컨테이너를 삭제하면서 그 안에 데이터베이스 정보가 날아갔을거라는 생각이 들었다.

#### 6단계 : MySQL 컨테이너 재설정

``` bash
# MySQL 접속 (비밀번호 입력)
docker exec -it mysql-container mysql -u root -p

# 데이터베이스 확인
SHOW DATABASES;

# 테이블 확인
USE 데이터베이스명;
SHOW tables;

# 데이터가 없다면, 다시 생성해주자...
CREATE 쿼리문 실행
```