---
title: Filter와 Interceptor의 차이점을 말해주세요.
parent: 서버
nav_order: -8
---

<div style="text-align: center; display: flex;
    flex-direction: column;
    align-items: center;">
    <h5>오늘의 질문</h5>
    <div style="color: black; background-color: #F0F3F5; border-radius: 5px; width: 50%; padding: 20px;">
    Filter와 Interceptor의 차이점을 말해주세요.
    </div>
</div>

---

### Filter

Filter 는 서블릿 컨테이너 수준에서 동작한다. 요청 및 응답의 전처리 및 후처리를 수행한다. 요청 로깅, 인증, 인코딩 설정, CORS 처리, 캐싱, 압축 등의 공통 기능을 구현하는 데 사용된다.

### Interceptor

Interceptor 는 스프링 MVC의 핸들러 수준에서 동작한다. Dispatcher Servlet 이 컨트롤러를 호출하기 전에 Interceptor 를 거친다.

### AOP

AOP 는 메서드 호출 전/후/예외 시점 때 동작한다. Interceptor 와 비슷한 동작을 하지만 개념적으로 다른데, AOP 는 핵심 로직에 부가 기능을 주입하는 프로그래밍 패러다임이다.

<br>

📎 참고자료

[매일메일](https://www.maeil-mail.kr/question/10)