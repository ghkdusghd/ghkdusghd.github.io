---
title: '@Component'
parent: Spring Boot
nav_order: -4
---

# 05. Component

> Bean을 따로 등록하지 않아도 @Component 어노테이션을 붙이면 자동으로 등록된다.

> &nbsp; 🥑 실습을 위한 아보카도 클래스 🥑
>
> Component 어노테이션은 클래스에 붙인다.
>
> <img src="/assets/images/pages/cs/spring/03. Component.png">

### 1️⃣ &nbsp;XML 파일에서의 사용법

1. context:component-scan 태그로 base-package 안에 입력한 패키지를 스캔하여 어노테이션을 분석한다.

2. 컴포넌트를 사용하지 않는 클래스들은 직접 Bean 을 등록해야 한다.

3. 컴포넌트를 사용한다 하더라도 xml 파일에서 Bean 을 등록할 수 있다.

```xml
<context:component-scan base-package="beans" />
```

### 2️⃣ &nbsp;JAVA 파일에서의 사용법

1. Configuration 파일에서 @ComponentScan 어노테이션과 패키지명을 적어넣으면 동작한다.

2. basePackages 에 적힌 패키지명을 스캔하여 컴포넌트를 동작시킨다.

```JAVA
@Configuration
@ComponentScan(basePackages = "beans")
public class BeanClass {
    ...
}
```

3. @Component 어노테이션을 적어넣은 클래스만 별도의 Bean 등록 없이 사용 가능하다.

```JAVA
@Component
@Lazy
public class Avocado {
	...
}
```
