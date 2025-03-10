---
title: 프로그래밍 기본
parent: 기초
nav_order: -1
---

# JDK, SDK, API, IDE ...도대체 뭔데 ??

> 프로그래밍 언어는 고급 언어와 저급 언어로 구분된다. 고급 언어란 사람이 이해할 수 있는 언어를 말하고, 저급 언어는 기계어에 가까운 언어를 말한다. 사람이 Java,C,Python 등의 고급 언어로 소스 파일을 작성하면 컴퓨터가 바로 이해할 수 없기 때문에 컴파일 (compile) 이라는 과정을 통해 컴퓨터가 이해할 수 있는 0과 1로 이루어진 기계어로 변환해야 한다. 컴파일은 빌드 과정의 단계에 불과하다. 지금부터 프로그램을 만들어 사용자에게 서비스하는 일련의 과정을 알아보자.

### 🧩 JDK (Java Development Kit)

- 자바 프로그램을 개발하고 실행하기 위해서는 먼저 Java SE(자바 표준 문법) 의 구현체인 JDK 를 설치해야 한다.

- JDK 는 자바 개발 키트의 약자로, 개발자들이 자바로 개발하는데 사용하는 SDK 키트이다.
  (SDK? Software Development Kit! 소프트웨어를 만들기 위한 라이브러리, 컴파일러, 디버거, 코드샘플 및 문서로 구성된 개발 키트를 말한다.)

- JDK 에는 Open JDK 와 Oracle JDK 가 있다. Oracle JDK 는 Open JDK 보다 응답성과 JVM 성능이 상대적으로 뛰어나다고 한다. 하지만 Open JDK 의 성능도 지속적으로 향상되고 있으며, 더욱 안정화되었기 때문에 JDK 비용을 고려한다면 Open JDK 를 사용하는 것이 유리하다.

### 🧩 API (Application Programming Interface)

- 모듈화하여 만들어진, 어떤 기능을 제어/제공하는 인터페이스를 말한다. 다시 말해 API 를 제공한다는 것은 기본 소프트웨어의 구성 요소의 특정 기능을 노출한다는 것이다.

- SDK 는 개발자가 직접 코드를 작성할 때 사용하는 도구이고, API 는 다른 개발자가 작성한 기능에 액세스할 때 사용한다.

### 🧩 IDE(Integrated Development Environment)

- 통합개발환경(IDE) 이란 프로그래머가 소프트웨어 코드를 효율적으로 개발하도록 돕는 소프트웨어 애플리케이션이다.

- IDE 에 원하는 JDK 버전을 설치하거나 Spring 같은 개발자의 편의를 위한 프레임워크를 설정하여 개발 환경을 세팅한다.

- 소프트웨어 편집, 빌드, 테스트, 패키징 같은 기능을 하나의 애플리케이션에 통합하여 개발자 생산성을 높인다.

### 🧩 Framework 와 Library

🍙 Framework ?

개발자가 소프트웨어를 개발함에 있어 코드를 구현하는 개발 시간을 줄이고, 코드의 재사용성을 증가 시키기 위해 일련의 클래스 묶음이나 뼈대, 틀을 라이브러리 형태로 제공하는 것을 말한다.

🍙 Library ?

개발자가 만든 클래스들을 말하며, 다른 프로그램들에서 사용할 수 있도록 제공하는 것을 말한다.

🍙 차이점 !

주요한 차이점은 제어의 흐름에 대한 주도성을 누가 갖고 있냐는 것이다. 프레임워크는 그 스스로가 제어 흐름의 주도성을 갖는 반면, 라이브러리의 제어의 흐름은 개발자가 제어한다.

### 🧩 Compile - Build(Packaging) - Deploy(배포)

```markdown
# 예시

내가 번역가라고 생각해보자.

이번에 출판사로부터 일을 하나 받았는데,

영문으로 된 글을 한국어로 번역하는 일이다.

번역된 글은 출판사에서 적절히 페이지를 나누어 책으로 엮을 것이며,

완성된 책은 출판이 되어 서점에 진열될 것이다.

우리는 이렇게 컴파일, 빌드, 배포의 모든 과정을 훑어본 것이다.

# 위의 과정을 좀 더 프로그래밍 관점에서 볼까 ?

1. 컴파일 : 개발자가 작성한 코드를 컴퓨터가 이해할 수 있는 언어로 번역하는 일.

2. 빌드 : 컴파일된 코드를 실제 실행할 수 있는 상태로 만드는 일 혹은 컴파일을 포함해 war, jar 등의 실행 가능한 파일을 뽑아내기까지의 과정을 빌드한다고 하기도 한다.

3. 배포 : 빌드가 완성된 실행 가능한 파일을 사용자가 접근할 수 있는 환경에 배치시키는 일.
```

🍙 Compile?

사용자가 작성한 코드를 컴퓨터가 이해할 수 있는 기계어로 변환하는 작업. 자바의 경우, 자바 언어로 작성한 소스 파일의 확장명은 .java 이다. 이후 javac(java compiler) 명령어는 소스 파일을 컴파일 하는데, 컴파일 결과 확장명이 .class 인 바이트코드 파일이 생성된다.

🍙 Build?

소스 코드 파일을 컴퓨터나 휴대폰에서 실행할 수 있는 독립 소프트웨어 가공물로 변환하는 과정을 말하거나 그에 대한 결과물을 일컫는다. 빌드 단계 안에 컴파일이 있다고 봐도 된다. 빌드를 한다면 소스 코드를 컴파일해서 .class 로 변환하고 resource 를 .class 에서 참조할 수 있는 적절한 위치로 옮기고 META-INF 와 MANIFEST.MF 들을 하나로 압축한다.

| 🔥 용어 정리                 | 설명                                                                                                                                                                                                                |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| JAR(Java Archive)            | Java 어플리케이션이 동작할 수 있도록 자바 프로젝트를 압축한 파일 형식이다. 클래스 파일과 리소스 및 메타 데이터를 포함한다. JRE 만 있어도 실행 가능하다.                                                             |
| WAR(Web Application Archive) | Jar 파일의 일종으로, 웹 애플리케이션 전체를 패키징하기위한 Jar 파일이다. 웹 관련 자원을 포함하며, 실행하려면 별도의 웹 서버(WEB) 혹은 웹 컨테이너(WAS) 가 필요하다.                                                 |
| 메타 데이터                  | 다른 데이터를 기술하기 위해 사용하는 데이터                                                                                                                                                                         |
| MANIFEST.MF                  | JAR 파일 자체에 대한 메타 데이터를 제공하는 파일이며, 빌드 후 Jar 파일을 압축 해제 해보면 META-INF 에 MANIFEST.MF 파일이 생성된다. (좀 더 쉽게 보자면 jar 파일의 사용매뉴얼이나 스펙을 기술한 사용설명서 느낌이다.) |
| META-INF                     | java 애플리케이션에서 사용되는 메타 데이터와 설정 정보들을 저장하는 폴더이다. (jar 든 war 든 META-INF 폴더를 가진다)                                                                                                |

🍙 Deploy?

빌드된 jar 파일(실행 가능한 파일)을 사용자가 접근할 수 있는 환경에 배치시키는 일을 말한다.

&nbsp;

🔖 참고문서

[AWS (SDK와 API의 차이)](https://aws.amazon.com/ko/compare/the-difference-between-sdk-and-api/)

[컴파일? 빌드? 배포? 개념과 차이는 무엇일까?](https://itholic.github.io/qa-compile-build-deploy/)

[MANIFEST.MF (티스토리)](https://codingdreamtree.tistory.com/108)
