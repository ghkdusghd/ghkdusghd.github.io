---
title: 시스템 콜이란 무엇인가요?
parent: 시스템
nav_order: -3
---

<div style="text-align: center; display: flex;
    flex-direction: column;
    align-items: center;">
    <h5>오늘의 질문</h5>
    <div style="color: black; background-color: #F0F3F5; border-radius: 5px; width: 50%; padding: 20px;">
    시스템 콜이란 무엇인가요?
    </div>
</div>

---

### 📍 정답 확인

운영체제는 사용자가 실행하는 프로그램이 하드웨어 자원에 직접 접근하는 것을 방지해 자원을 보호한다. 왜냐하면 프로그램이 CPU, 메모리, 하드 디스크에 마음대로 접근하고 조작할 수 있다면 자원이 무질서하게 관리될 수 있고, 한 프로그램의 실수가 전체 컴퓨터에 영향을 주기 때문이다. 운영체제는 프로그램들이 자원에 접근하려 할 때 오직 자신을 통해서만 접근하도록 하여 자원을 보호한다.

<br>

### 사용자 모드 (User Mode)

운영체제 서비스를 제공받을 수 없는 모드이다. CPU 가 해당 모드인 경우 입출력 명령어와 같은 하드웨어 자원 접근 명령을 실행할 수 없다. 일반 프로그램은 기본적으로 사용자 모드로 실행된다.

<br>

### 커널 모드 (Kernel Mode)

운영체제 서비스를 제공받을 수 있는 실행 모드로, 커널 영역의 코드를 실행할 수 있다.

<br>

### 시스템 콜 (System Call)

사용자 모드로 실행된 프로그램이 자원에 접근하는 운영체제 서비스를 제공받으려면 운영체제게 요청을 보내 커널 모드로 전환되어야 한다. '시스템 콜' 은 이때 운영체제 서비스를 제공받기 위한 요청을 말한다. 시스템 콜은 일종의 소프트웨어 인터럽트이다. 시스템 콜을 발생시키는 명령어가 실행되면 CPU 는 현재까지의 작업을 백업한 뒤 커널 영역 내에 시스템 콜을 수행하는 인터럽트 서비스 루틴을 실행한 이후, 다시 기존 실행하고 있던 프로그램으로 복귀하여 실행을 계속한다.

<br>

### 🤔 컴퓨터의 구조가 궁금해졌다...!

컴퓨터는 3층 집으로 구성되어 있다. 1층 H/W, 2층 S/W, 3층 Application 계층으로 생각하면 편하다. 2층에 위치한 OS 는 3층의 프로세스들을 지원(support)해주고, 1층의 자원들을 관리한다. 개발자가 InteliJ 같은 애플리케이션을 활용해서 "Hello World" 를 화면에 출력하는 코드를 실행시킨다고 가정해보자. 컴퓨터는 아래 그림과 같이 동작한다.

<img src="/assets/images/pages/cs/maeil-mail/system/스크린샷 2025-02-05 오전 10.54.21.png">

1. 개발자가 키보드라는 주변기기로 "Hello World" 출력을 요청한다. 이것을 Interrupt 라고 한다. (주변기기에 의한 입출력 요청)

2. API 가 개발자의 출력 요청을 운영체제로 전달한다.

3. 운영체제는 요청에 필요한 wirte() 함수를 실행시키는데, 사용자 모드에서 커널 모드로 전환시키는 것이 시스템 콜이다.

4. H/W 로 전달된 요청은 드라이버가 받아 실행시키는데, 이것을 IRQ (인터럽트 요청) 라고 한다. PC 내부의 장치들은 어떤 장치가 몇 번 IRQ 를 사용할지 고유한 값을 가지는데, 요청에 해당하는 IRQ 를 드라이버가 찾아 실행시킨다고 보면 된다.

5. Hello World 를 출력하기 위해 Video 장치가 실행되었고, 작업이 완료되었음을 리턴한다.

<br>

🔖 참고자료

[매일메일](https://www.maeil-mail.kr/question/154)

[컴퓨터가 3층집인건 알고 있죠?](https://www.youtube.com/watch?v=M9ZrQX1UgAU)