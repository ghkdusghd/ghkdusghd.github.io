---
title: Bean Validation
parent: Spring MVC
nav_order: -15
---

# 13. Bean Validation

> 입력값이 공란이면 안되거나 입력할 수 있는 숫자의 범위가 제한되어 있는 등, 검증이 필요한 항목은 사실 어느 서비스나 비슷할 것이다. 그렇기 때문에 스프링은 기본적인 검증 항목들을 어노테이션으로 제공하고 있다. 이것을 활용한 검증 방식을 Bean Validation 이라고 한다.

### 1️⃣ &nbsp; Bean Validation - FieldError

- Item 객체에 다음과 같이 @ 어노테이션을 추가한다.
- 검증 로직을 수행할 @ModelAttribute 앞에 @Validated 어노테이션을 추가한다.

```java

public class Item {

   private Long id;

  @NotBlank //공백, 띄어쓰기를 허용하지 않음.
   private String itemName;

  @NotNull //null을 허용하지 않음.
  @Range(min = 1000, max = 1000000) //입력 가능한 숫자 범위를 지정.
   private Integer price;

  @NotNull
  @Max(value = 9999) //입력 가능한 최대값을 지정.
  Integer quantity;

}

```

```java
@PostMapping("/add")
public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {
    (...)
}
```

### 2️⃣ &nbsp; errors.properties

- Bean Validation 으로 출력되는 에러 메시지를 좀 더 자세히 작성하고 싶다면 properties 파일에 메시지를 등록하면 된다.

```properties
#Bean Validation 에러 메시지 추가
NotBlank={0} 공백X
Range={0}, {2} ~ {1} 허용
Max={0}, 최대 {1}
```

### 3️⃣ &nbsp; Bean Validation - ObjectError

- 특정 필드(FieldError) 가 아닌 오브젝트 관련 오류(ObjectError) 를 처리하려면 @ScriptAssert() 어노테이션을 사용하면 된다.

```java
@ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 10000", message = "총합계는 10,000원 이상이어야 합니다.")
public class Item {
   (...)
}
```

- 그렇지만 실제 사용해보면 제약이 많고 복잡하기에, ObjectError의 경우 자바 코드로 작성하는 것을 권장한다.

### 4️⃣ &nbsp; groups

- Bean Validation은 이렇게 편리한 기능을 제공해주지만 등록 폼과 수정 폼에서의 검증 요구사항이 다른 경우 적용하기 힘들어진다.
- 이 경우 방법은 두 가지인데, 첫번째로 groups 기능을 활용할 수 있다.
- 객체에 적용한 Bean Validation 어노테이션에 해당 검증사항을 적용할 클래스들을 적어넣어 그룹화한다.
- 컨트롤러의 @Validated 어노테이션에 어떤 그룹의 검증 항목을 적용할 것인지를 적어넣는다.

```java

public class Item {

   @NotNull(groups = UpdateCheck.class)
    private Long id;

   @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
    private String itemName;

   @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
   @Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class})
    private Integer price;

   @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
   @Max(value = 9999, groups = {SaveCheck.class})
    private Integer quantity;

}

```

```java
@PostMapping("/add")
public String addItemV6(@Validated(SaveCheck.class) @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {
    (...)
}
```

### 5️⃣ &nbsp; 폼 객체 생성

- 위 groups 방식을 사용하면 코드의 복잡도가 증가한다.
- 검증 항목을 분리해야 하는 이유는 등록 폼과 수정 폼에서 전송하는 필드가 다르기 때문이다.
- 이 경우 등록, 수정 폼에서 데이터를 전달받을 객체를 따로 생성해서 @ModelAttribute 에 담은 후 검증을 수행한 이후 Item 객체에 데이터를 세팅하면 간단해진다.

```java

public class ItemSaveForm {

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
    @Max(value = 9999)
    private Integer quantity;

}
```

```java

public class ItemUpdateForm {

    @NotNull
    private Long id;

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    //[수정] 에서 수량은 자유롭게 변경할 수 있다.
    private Integer quantity;

}

```

```java
/* 컨트롤러 */
public String edit(@PathVariable Long itemId, @Validated @ModelAttribute("item") ItemUpdateForm form, BindingResult bindingResult) {
}
```
