---
title: "[Vercel] Vercel 로 리액트 프로젝트 배포하기"
parent: NCBT
nav_order: -9

tags:
  - [vercel, react]

toc: true
toc_sticky: true

date: 2024-12-23
last_modified_at: 2024-12-23
---

> 지난 시간에는 네이버클라우드 + 도커로 스프링 부트 애플리케이션을 배포해보았다. 오늘은 Vercel 로 리액트 애플리케이션을 배포해보자 !

> ### 1️⃣ Vercel 로 배포하기

우선 프론트엔드 애플리케이션은 백엔드 보다 용량이 훨씬 크다. 때문에 백엔드 서버와 동일하게 네이버 클라우드로 배포한다면... 어마어마한 비용이 부과될 것으로 예상되어, 무료 서비스인 Vercel 을 이용하기로 했다.

#### 1단계 : [Vercel 회원가입 (클릭하면 이동)](https://vercel.com/)

- 가입 후 배포할 깃허브 리포지토리를 연동한다. Import 버튼을 누르고 다음으로 넘어가자.

<img width="524" alt="스크린샷 2024-12-23 오후 3 04 51" src="https://github.com/user-attachments/assets/cca7549d-050a-4e24-8cd3-4d9b96b9ee1f" />

#### 2단계 : 프로젝트 환경설정

- Build and Output Settings
    - Build Command : 프로젝트를 빌드하기 위해 실행되는 커맨드로, 기본설정으로 진행해도 배포가 잘 된다.

- Environment Variables
    - 빌드 및 배포 시 필요한 환경변수들을 넣어준다. env 파일을 복붙해도 자동완성 해준다.

<img width="689" alt="스크린샷 2024-12-23 오후 3 10 31" src="https://github.com/user-attachments/assets/0325c904-2b6f-413c-a21e-d061b0f1592b" />

#### 3단계 : 배포

- 위 설정을 마친 후 Deploy 버튼을 누르면 자동적으로 배포가 완료된다!

<img width="664" alt="스크린샷 2024-12-23 오후 3 16 13" src="https://github.com/user-attachments/assets/426bb669-c9cf-45a9-aa98-55286af582f4" />

