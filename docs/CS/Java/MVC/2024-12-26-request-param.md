---
title: Request Parameter
parent: Spring MVC
nav_order: -11
---

# 09. Request Parameter

> 스프링으로 HTTP 요청 데이터를 조회하는 방법을 알아보자.

### 1️⃣ &nbsp; 클라이언트에서 서버로 요청 데이터를 전달하는 방식

- 클라이언트가 데이터를 전달하는 방식에 따라 조회하는 방법이 달라질 것이다. 그러므로 먼저 전달 방식을 알아보자. 크게 아래 3가지 방법으로 나뉜다.

- 요청 파라미터 방식과 HTML Form 을 사용한 전달방식은 같은 방법으로 데이터를 조회할 수 있다.

| 전달 방식     | 설명                                         | 상세                                                                                         | 조회 메서드                    |
| ------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------ |
| 요청 파라미터 | URL의 쿼리 파라미터에 데이터를 포함해서 전달 | http://localhost:8080/request-param-v2?username=hello&age=10                                 | @RequestParam, @ModelAttribute |
| HTML Form     | 메시지 바디에 쿼리 파라미터 형식으로 전달    | 클라이언트가 HTML Form 에서 값을 입력하고 전송 버튼을 누를 때 데이터가 넘어오는 것을 말한다. | 上同                           |
| HTTP API      | message body 에 직접 데이터를 담아서 전달    | 주로 JSON형식을 사용한다.                                                                    | HttpEntity<<T>T>, @RequestBody |

### 2️⃣ &nbsp; 요청 파라미터 - @RequestParam

- 파라미터 이름으로 바인딩해준다. 변수명이 파라미터 이름과 같다면 name 속성을 생략해도 된다.

- 파라미터를 Map, MultiValueMap 으로 조회할 수 있다. 이렇게 하면 모든 요청 파라미터가 Map 에 담긴다. 다만 하나의 key에 여러 개의 데이터가 담길 경우를 상정한다면 (key=userIds, value=[id1,id2]) MultiValueMap 을 사용해야 한다.

```java
@Slf4j
@Controller
public class RequestParamController {

    @ResponseBody
    @RequestMapping("/request-param-v3")
    public String requestParamV3(@RequestParam String username,
                                 @RequestParam int age)
    {
        log.info("username = {}, age = {}", username, age);
        return "ok";
    }

    @ResponseBody
    @RequestMapping("request-param-map")
    public String requestPatamMap(@RequestParam Map<String, Object> paramMap) {
        log.info("username = {}, age = {}", paramMap.get("username"), paramMap.get("age"));
        return "ok";
    }

}
```

### 3️⃣ &nbsp; 요청 파라미터 - @ModelAttribute

- 실제 개발을 하면 요청 파라미터를 받아서 객체에 그 값을 넣어줘야한다. 이 과정을 자동화한 것이 @ModelAttribute 이다.

- @ModelAttribute 어노테이션을 아예 생략해도 되지만 일반적으로 권장되지는 않는 모양이다.

- 우선 테스트로 사용할 HelloData 클래스를 만든다.

```java

import lombok.Data;

@Data
public class HelloData {
    private String username;
    private  int age;
}

```

- @ModelAttribute 어노테이션을 사용해서 바로 객체에 값을 주입할 수 있다.

```java
@Slf4j
@Controller
public class RequestParamController {

    @ResponseBody
    @RequestMapping("/model-attribute-v1")
    public String modelAttributeV1(@ModelAttribute HelloData helloData) {
        log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());
        log.info("heloData = {}", helloData);
        return "ok";
    }

}
```

#### ⚡️ &nbsp; @ModelAttribute 의 특별한 사용법 !

- @ModelAttribute 어노테이션을 메소드 단위에 사용하면, 해당 컨트롤러를 요청할 때, 메소드의 return 값이 Model에 자동으로 담기게 된다.

```java
@Controller
public class FormItemController {

    @ModelAttribute("regions")
    public Map<String, String> regions() {
        Map<String, String> regions = new LinkedHashMap<>();
        regions.put("SEOUL", "서울");
        regions.put("BUSAN", "부산");
        regions.put("JEJU", "제주");
        return regions;
    }

}
```

### 4️⃣ &nbsp; HTTP API (단순 텍스트) - HttpEntity<<T>T>

- 요청 파라미터와 다르게 HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우는 @RequestParam, @ModelAttribute 를 사용할 수 없다.

- 그래서 스프링MVC 에서는 HTTP header, body 정보를 간편하게 조회할 수 있는 HttpEntity 라는 파라미터를 제공한다.

- HttpEntity 가 제공하는 getBody, getHeader 등의 메서드를 통해 HTTP 정보를 얻어낼 수 있다.

- 메시지 컨버터가 HTTP header 의 Content-Type 을 보고 해당하는 타입으로 자동으로 변환해준다.

```java
@Slf4j
@Controller
public class RequestBodyStringController {

    @PostMapping("/request-body-string-v3")
    public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {

        String messageBody = httpEntity.getBody();
        log.info("messagebody = {}", messageBody);

        return new HttpEntity<>("ok");
    }

}
```

### 5️⃣ &nbsp; HTTP API (단순 텍스트) - @RequestBody

- HttpEntity 를 더 단순하게 사용하기 위한 어노테이션이 있다.

- @RequestBody 는 HTTP 의 바디 정보만을 가져올 수 있다.

```java
@Slf4j
@Controller
public class RequestBodyStringController {

    @ResponseBody
    @PostMapping("request-body-string-v4")
    public String requestBodyStringV4(@RequestBody String messageBody) {
        log.info("messagebody = {}", messageBody);
        return "ok";
    }

}
```

### 6️⃣ &nbsp; HTTP API (JSON)

- JSON Type 으로 넘어오는 데이터 역시 HttpEntity<<T>T>, @RequestBody 방식으로 조회할 수 있다.

```java
@Slf4j
@Controller
public class RequestBodyJsonController {

    @ResponseBody
    @PostMapping("request-body-json-v4")
    public String requestBodyJsonV4(HttpEntity<HelloData> helloData) {
        HelloData data = helloData.getBody();
        log.info("username = {}, age = {}", data.getUsername(), data.getAge());
        return "ok";
    }

    @ResponseBody
    @PostMapping("/request-body-json-v3")
    public String requestBodyJsonV3(@RequestBody HelloData helloData) {
        log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }

}
```
