---
title: Week6 - RESTful 아키텍처
parent: Spring6
nav_order: 1
---

# 목차

1. RESTful 아키텍처 원칙
- REST 아키텍처 스타일의 제약조건
- API 설계 best practices

2. Spring REST 컨트롤러 심화
- @RestController 어노테이션의 세부 기능
- @RequestBody와 @ResponseBody 상세 사용
- HTTP 메서드에 따른 적절한 어노테이션 사용 (GET, POST, PUT, DELETE)

3. 응답 상태 코드와 에러 처리
- ResponseEntity 확장을 통한 Response 처리 전략
- HTTP 상태 코드의 적절한 사용
- @ExceptionHandler와 @ControllerAdvice를 이용한 예외 처리

4. 페이징과 정렬, 검색조건 처리
- 페이징 처리
- 정렬 처리
- 단어 검색
- Specification을 통한 동적 쿼리 작성과 코드 재사용

5. 문서화와 호출 테스트
- Spring REST Doc 혹은 OpenAPI를 이용한 문서 작성
- cURL, RestClient 활용한 API 호출 테스트
