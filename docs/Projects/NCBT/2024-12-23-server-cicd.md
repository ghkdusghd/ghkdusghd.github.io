---
title: "[NaverCloud + Docker + Jenkins] 젠킨스로 CI/CD 구현하기"
parent: NCBT
nav_order: 10

tags:
  - [NCP, jenkins, docker]

toc: true
toc_sticky: true

date: 2024-12-23
last_modified_at: 2024-12-23
---

### 1️⃣ 클라우드 서버에 젠킨스 실행

``` bash
# 도커 이미지가 없다면 자동으로 다운받아 실행시킵니다.
docker run \
  --name jenkins \
  -p 8081:8080 \
  -e TZ=Asia/Seoul \
  -v /Documents/Projects/jenkins/var/jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /Documents/Projects/jenkins/data:/data \
  -u root \
  -d \
  --restart unless-stopped \
  jenkins/jenkins:lts


# 이미지 있는지 확인 (jenkins/jenkins 라는 이미지가 있어야 함)
docker images

# 컨테이너 실행중인지 확인
docker ps

# 초기 비밀번호 확인합시다!
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### 2️⃣ 젠킨스에 접속

- 브라우저에서 ```공인IP:8081``` 포트로 접속 후,

- 아래 화면에 초기 비밀번호를 입력하기

<img width="667" alt="스크린샷 2024-12-23 오전 10 49 12" src="https://github.com/user-attachments/assets/10d836c3-df71-4a6a-9d8e-d353765b243f" />

<br>
<br>

- 계속 진행...

<img width="667" alt="스크린샷 2024-12-23 오전 10 50 16" src="https://github.com/user-attachments/assets/f01aed41-2ad0-4867-b4b9-26747ba27db0" />

<img width="682" alt="스크린샷 2024-12-23 오전 10 52 12" src="https://github.com/user-attachments/assets/b21f6841-7562-474a-90e5-f6448a1dd702" />

<br>
<br>

- 관리자 계정을 생성하고 Jenkins URL 은 그대로 두고 진행 !

<img width="678" alt="스크린샷 2024-12-23 오전 10 54 35" src="https://github.com/user-attachments/assets/3e8e3557-aa9e-4124-ab36-d906a57ffb45" />

### 3️⃣  깃허브 저장소 클론


#### 1단계 : 프로젝트 디렉토리 생성 후 이동

``` bash
# 프로젝트를 클론할 로컬 저장소를 만들자
sudo mkdir -p /Documents/Projects/프로젝트명

# 이동
cd /docker_projects/프로젝트명
```

#### 2단계 : 원격 저장소와 연결

``` bash
git init

# initial branch 가 'master' 라면 'main' 으로 바꾸자
git config --global init.defaultBranch main

# 깃 클론
git remote add origin '복제한 깃 레포지토리 주소'

# 원격저장소와 잘 연결되었는지 확인
git remote -v

# 저장소에서 pull 받아오기
git pull origin main
```

#### 3단계 : 권한 설정

``` bash
# gradlew 권한 설정
chmod 744 ./gradlew

# java 설치 확인
java -version

# 설치되어있지 않다면 다운받자
sudo apt update
sudo apt install openjdk-17-jdk
```

#### 4단계 : 환경변수 파일 넣기

``` bash
# 아래 경로에 application.yml 파일 생성
vi src/main/resources/application.yml

# 프로젝트에 필요한 환경변수를 복사해서 붙여넣고 저장
i # 눌러 편집
esc # 편집 종료
:wq # 저장하고 종료
:q! # 저장하지 않고 종료 
```

#### 5단계 : 빌드 실행 및 도커 이미지 생성

``` bash
# 테스트 생략하고 빌드 실행
./gradlew clean build -x test

# 완료 후 아래 경로에 .jar 파일이 생기는지 확인
cd build/libs

# 
```