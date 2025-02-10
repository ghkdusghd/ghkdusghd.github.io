---
title: URI, URL, URN의 차이점은 무엇인가요?
parent: 웹
nav_order: -3
---

<div style="text-align: center; display: flex;
    flex-direction: column;
    align-items: center;">
    <h5>오늘의 질문</h5>
    <div style="color: black; background-color: #F0F3F5; border-radius: 5px; width: 50%; padding: 20px;">
    URI, URL, URN의 차이점은 무엇인가요?
    </div>
</div>

---

<!-- ### ✏️ 나의 답변
``` markdown
인터넷 링크 전체를 URL 이라고 하고, 그 세부적으로 URL 과 URI 로 나뉜다.
URI - 도메인 + 하위구조
URN - 도메인 제외한 하위 구조
```

<br> -->

### 정답 확인

<img src="/assets/images/pages/cs/maeil-mail/web/스크린샷 2025-02-07 오후 12.21.03.png">

<br>

### URI

인터넷에서 자원을 식별하기 위한 문자열을 말하며, URL + URN 을 포함한 상위 개념이다.

<br>

### URL

URI 의 한 형태로, 인터넷에서 자원의 위치를 나타내는 방식이다. 자원이 어디에 있는지를 설명하는데 사용되며, 자원에 접근하기 위한 프로토콜을 포함한다. 예를 들자면 아래와 같다.

``` markdown
https://www.example.com/path/to/resource
```

<br>

### URN

자원의 위치와 관계 없이 자원의 이름을 식별하는 방식이다. 자원의 위치가 변하더라도 동일한 식별자를 유지할 수 있게 한다. 특정한 스키마를 따르며, 자원에 대한 영구적인 식별자를 제공한다.

``` markdown
urn:isbn::0451450523 (특정 책의 isbn 번호)
```

<br>

🔖 참고 자료

[매일메일](https://www.maeil-mail.kr/question/149)