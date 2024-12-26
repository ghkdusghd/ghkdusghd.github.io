---
title: Request Mapping
parent: Spring MVC
nav_order: -9
---

# 08. Request Mapping

> URL 요청을 처리할 컨트롤러를 매핑할 때 쓰는 스프링 기능들을 알아보자.

#### &nbsp; &nbsp; 🙋🏻‍♀️ &nbsp; 들어가기 전에, @Controller 와 @RestController 란?

&nbsp; &nbsp; 💡 스프링에서는 클래스 단위에 @Controller 어노테이션을 붙여두면 컨트롤러 역할을 하는 클래스로 인식한다 !

| 어노테이션      | 설명                                                                         |
| --------------- | ---------------------------------------------------------------------------- |
| @Controller     | 메서드의 반환값이 String이면 뷰 네임으로 인식한다.                           |
| @RestController | 반환 값으로 뷰를 찾는 것이 아닌, HTTP 메시지 바디에 반환 값을 바로 입력한다. |

### 1️⃣ &nbsp; HTTP 메서드 매핑

| 어노테이션      | 설명                                                                                                                   | 종류                                                                  |
| --------------- | ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| @RequestMapping | (value) 에 들어가는 URL경로와 메서드를 매핑한다. HTTP 메서드를 축약한 어노테이션을 사용하는 것이 더 직관적이다.        | @GetMapping, @PostMapping, @PutMapping, @DeleteMapping, @PatchMapping |
| @PathVariable   | URL에 요청 파라미터로 넘어오는 값을 읽어들일 수 있다. 변수명과 파라미터명이 같으면 ()안의 파라미터명을 생략할 수 있다. | -                                                                     |

#### &nbsp; 🌱 &nbsp; @RequestMapping

```java
@RestController
public class MappingController {

    @RequestMapping(value = "/hello-basic", method = RequestMethod.GET)
    public String helloBasic() {
        log.info("helloBasic");
        return "ok";
    }

}

@RestController
@RequestMapping("/mappings/users")
public class MappingClassController {

    @GetMapping
    public String users() {
        return "get users";
    }

    @PostMapping
    public String addUser() {
        return "post user";
    }

}

```

#### &nbsp; 🌱 &nbsp; @PathVariable

```java
@RestController
public class MappingController {

    @GetMapping("/mapping/{userId}")
    public String mappingPath(@PathVariable("userId") String data) {
        log.info("mappingPath userId = {}", data);
        return "ok";
    }

    //다중사용도 가능하다.
    @GetMapping("/mapping/{userId}/orders/{orderId}")
    public String mappingPath(@PathVariable String userId, @PathVariable String orderId) {
        log.info("mappingPath userId = {}, orderId = {}", userId, orderId);
        return "ok";
    }

}
```
