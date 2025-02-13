---
title: 연결 리스트에 대해서 설명해주세요.
parent: 자료구조
nav_order: -2
---

<div style="text-align: center; display: flex;
    flex-direction: column;
    align-items: center;">
    <h5>오늘의 질문</h5>
    <div style="color: black; background-color: #F0F3F5; border-radius: 5px; width: 50%; padding: 20px;">
    연결 리스트에 대해서 설명해주세요.
    </div>
</div>

---

### 연결 리스트 (Linked List)

리스트 내의 요소(노드)들을 포인터로 연결하여 관리하는 선형 자료구조이다. 각 노드는 데이터와 다음 요소에 대한 포인터를 가지며, 이 때 첫 번째 노드를 HEAD, 마지막 노드를 TAIL 이라고 한다. 연결 리스트는 메모리가 허락하는 한 요소를 계속 삽입할 수 있으며, 시간 복잡도는 탐색에는 O(n), 노드 삽입과 삭제는 O(1) 라는 특징을 가지고 있다. 파생된 자료구조는 단일 연결 리스트 (Singly Linked List), 이중 연결 리스트 (Doubly Linked List, Circular Linked List) 가 있다.

배열은 순차적인 데이터가 들어가기 때문에 메모리 영역을 연속해서 사용하지만, 연결 리스트는 메모리 공간에 흩어져서 존재한다는 점에서 차이가 있다.

<img src="/assets/images/pages/cs/maeil-mail/structure/스크린샷 2025-02-13 오후 1.16.43.png">

<br>

🔖 참고자료

[매일메일](https://www.maeil-mail.kr/question/163)

[널널한 개발자TV - 단일 연결 리스트 구현 첫 번째](https://www.youtube.com/watch?v=i_rONJmWeKY)