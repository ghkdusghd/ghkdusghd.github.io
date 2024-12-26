---
title: Properties
parent: Spring MVC
nav_order: -10
---

# 03. properties

> 외부에서 특정 값들을 주입받아야 할 때, (DB접속 정보 등) 이를 객체별로 코딩해두면 변경이 있을 때 처리하기 힘들어지는 단점이 있다. 그렇다고 public 저장소에 저장하면 외부에 노출될 가능성도 있기에 중요한 값들은 properties 라는 파일에 작성해두고 필요할때 값을 주입하는 방법을 사용한다.

### 1️⃣ &nbsp;properties 파일 작성

1.  /WEB-INF/properties 경로에 확장자명을 .properties 로 생성한다.

2.  "어노테이션명.클래스명.필드명 = 값" 의 형식으로 적는다.

```properties
#data1.properties
class.num1 = 10
class.string1 = 문자열1
```

### 2️⃣ &nbsp;작성한 properties 파일을 주입

1.  작성한 properties 파일을 읽어오기 위해 ReloadableResourceBundleMessageSource (이하 res) 를 [ServletAppContext] 클래스에 스프링 빈으로 등록한다.

```java
public class ServletAppContext implements WebMvcConfigurer{

    //Bean name = "프로퍼티명"
	@Bean(name = "messageSource")
	public ReloadableResourceBundleMessageSource source() {
		ReloadableResourceBundleMessageSource res = new ReloadableResourceBundleMessageSource();
	    res.setBasename("/WEB-INF/properties/data1");

	    return res;
	}

}
```

2. 등록한 빈은 Controller에서 자동주입 받는다.

```java
public class SpringController {
	@Autowired
	ReloadableResourceBundleMessageSource res;
}
```

### 3️⃣ &nbsp;properties 값을 읽어오기

1. res 의 getMessage 메서드로 프로퍼티 값을 읽어올 수 있다.

```java
public String t1(Model model, Locale locale) {

	String str1 = res.getMessage("class.num1", null, null);
	String str2 = res.getMessage("class.string1", null, null);
 }
```
