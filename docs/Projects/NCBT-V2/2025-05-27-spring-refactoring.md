---
title: "[NCBT-V2] Spring 코드 최적화"
parent: NCBT-V2
nav_order: -2

date: 2025-05-27
---

# <span style="color: #7153ED; font-weight: bold;">NCBT V2 </span> Spring 코드 최적화

NCBT V1 을 만들고 난 이후 스프링 그룹 스터디에서 공부하게 되었다. 거기서 얻은 지식과 경험으로 조금 더 좋은 프로그램을 짤 수 있을 거라 생각하기에 V2 버전에서는 코드 최적화를 염두에 두고 있었다. 아래 3가지 부분에서 최적화를 해보려고 한다.

1. RestClient 사용 : 기존에 썼던 RestTemplate 은 사용법이 직관적이지 않고 Template 클래스에 너무 많은 HTTP 기능이 노출된다는 단점이 있었다. 이를 보완하여 만든 것이 WebClient 이지만 논블로킹 방식으로 동작하기에 스프링에서 사용하기에 적절치 않다고 생각했고, 그 대안으로 RestClient 를 써보기로 했다.

2. JPA 마이그레이션 : 기존에는 MyBatis 로 SQL 매핑을 했는데, 사실 우리 서비스는 단순한 조회, 등록, 삭제 작업만 하는 정도의 서비스이다. 복잡한 쿼리문이 들어가지 않기 때문에 오히려 JPA 를 사용하는 것이 유지보수가 더 쉬워질 것 같았다.

3. Java mail 최적화 : 같이 플젝을 진행하던 언니가 취업하게 되어 나 혼자 V2 버전을 준비하게 되었다. 프로젝트에서 쓰였던 Java mail 은 언니 이메일로 설정했기 때문에 내 이메일로 다시 설정해야 할 것 같고, 기존에 mail 발송 시간이 상당히 느렸기 때문에 이 부분도 개선해보고 싶다.

---

## 1️⃣  RestClient 사용

RestTemplate 대신 RestClient 를 사용하여 가독성을 높였다.

- 변경 전

``` java
// 인가 코드로 액세스 토큰 요청
String getTokenUrl = "https://nid.naver.com/oauth2.0/token?grant_type=authorization_code"
        + "&client_id=" + naverClientId + "&client_secret=" + naverClientSecret
        + "&code=" + code + "&state=" + state;
Map<String, String> naverToken = restTemplate.getForObject(getTokenUrl, Map.class);
String naverAccessToken = naverToken.get("access_token");

// 접근 코드로 사용자의 로그인 정보 획득
String getUserUrl = "https://openapi.naver.com/v1/nid/me";
HttpHeaders headers = new HttpHeaders();
headers.set("Authorization", "Bearer " + naverAccessToken);
HttpEntity<String> request = new HttpEntity<>(null, headers);
ResponseEntity<Map> naverResponse = restTemplate.exchange(getUserUrl, HttpMethod.GET, request, Map.class);
Map<String, String> naverUser = (Map) naverResponse.getBody().get("response");
```

- 변경 후 

``` java
// 인가 코드로 액세스 토큰 요청
String getTokenUrl = "https://nid.naver.com/oauth2.0/token";
Map<String, String> naverToken = restClient.get()
        .uri(uriBuilder -> uriBuilder
                .path(getTokenUrl)
                .queryParam("grant_type", "authorization_code")
                .queryParam("client_id", naverClientId)
                .queryParam("client_secret", naverClientSecret)
                .queryParam("code", code)
                .queryParam("state", state)
                .build())
        .retrieve()
        .body(Map.class);
String naverAccessToken = naverToken.get("access_token");

// 접근 코드로 사용자의 로그인 정보 획득
String getUserUrl = "https://openapi.naver.com/v1/nid/me";
Map<String, Object> naverResponse = restClient.get()
        .uri(getUserUrl)
        .headers(headers -> headers.setBearerAuth(naverAccessToken))
        .retrieve()
        .body(Map.class);
Map<String, String> naverUser = (Map<String, String>) naverResponse.get("response");
```

## 2️⃣  JPA 마이그레이션

### 엔티티 설계

JPA 는 엔티티를 중심으로 객체지향적인 설계를 할 수 있다는 것이 장점이다. 즉 엔티티와 그 연관 관계를 설정하는 것이 중요하다. 현재 V2 모델에서는 V1 과 비교하여 DB에 변경사항이 있으므로 ERD 를 새로 작성하면서 연관관계까지 설정해 보았다.

<img src="/assets/images/pages/projects/ncbt-v2/스크린샷 2025-05-30 오후 11.19.36.png">

- 사용자는 점수를 기록할 수 있다. (1 : N)
- 사용자는 북마크를 저장할 수 있다. (1 : N)
- 사용자는 문제 오류를 신고할 수 있다. (1 : N)

scores, bookmarks, complaints 테이블이 N에 해당하므로 연관관계의 주인이 된다. 또한 기존에 MySQL 에 저장했던 refresh token 과 email 인증 코드는 Redis 에서 관리하는 것이 좋을 것 같아서 데이터를 이전하기로 했다.

## 3️⃣ 코드 최적화

1. ORM 으로 마이그레이션 진행

2. 조인, 서브쿼리가 들어가는 복잡한 쿼리는 MyBatis 매퍼를 그대로 사용했다.

3. @ControllerAdvice, @ExceptionHandler 를 추가하여 예외 처리 로직을 공통화했고, 비즈니스 로직과 예외 처리 로직을 분리함으로써 가독성과 유지보수성을 높였다.

4. Api 응답 DTO 를 도입하여 모든 API에서 일관된 응답을 제공하도록 개선. 클라이언트 측 처리를 단순화했다.

        @Getter @Setter
        public class ApiResult<T> {
        private T data;
        private HttpStatus status;
        private String message;
        private LocalDateTime dateTime;
        }

위의 개선 사항을 반영하여 V2 버전을 완성했다. 다만 자바 메일의 경우 전송 속도가 체감 3~5초 정도로 굉장히 느린 것이 문제였는데, 개발 당시에는 후순위로 미뤄두고 작업을 했어서 이번 기회에 개선해보고자 한다. 이 부분은 다음 포스팅으로 넘기겠다.
