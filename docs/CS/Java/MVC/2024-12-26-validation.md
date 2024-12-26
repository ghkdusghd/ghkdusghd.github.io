---
title: Validation
parent: Spring MVC
nav_order: -14
---

# 12. 검증 (Validation)

> 🤔 사용자가 웹페이지에 상품을 등록하고 수정할 수 있는 <상품 관리 시스템>을 개발했다고 가정해보자. 여기에 다양한 요구사항이 생겼다.
>
> 1. 타입 검증
>
> - 가격, 수량에 문자가 들어가면 검증 오류 처리
>
> 2. 필드 검증
>
> - 상품명 : 필수, 공백 X
> - 가격 : 1,000원 이상 ~ 1,000,000원 이하
> - 수량 : 최대 9,999개까지 등록 가능
>
> 3. 특정 필드의 범위를 넘어서는 검증.
>
> - 가격 \* 수량 = 10,000원 이상
>
> 좋은 웹 서비스는 이러한 오류가 발생하면 오류가 발생한 값을 유지한 상태로 어떤 오류가 발생했는지 사용자에게 친절히 알려주어야 한다.

<img src="/assets/images/pages/cs/mvc/08. Validation.png">
<span style="color: #808080">출처: 김영한의 스프링MVC (인프런)</span>

### 자, 이제 개발해보자 🙋🏻‍♀️

### 1️⃣ &nbsp; BindingResult

- 스프링이 제공하는 기능이다.
- 검증 오류를 보관하는 객체로, 오류가 발생하면 오류내용을 저장해준다.
- BindingResult 가 있으면 @ModelAttribute 에 데이터 바인딩 시 오류가 있어도 컨트롤러를 호출해준다.
- 다만, @RequestBody 로 데이터 바인딩을 하는 JSON 데이터의 경우 타입 에러가 발생하면 모델 객체에 데이터가 담기지 않고, 컨트롤러를 호출할 수 없게 된다. 이 경우는 예외 처리로 오류 메시지를 출력해야 하는데 이것은 다음 강의에서 자세히 알아보자.

### 2️⃣ &nbsp; BindingResult - rejectValue(), reject() 메서드

- BindingResult 는 rejectValue(), reject() 메서드를 제공한다. 필드명과 에러코드를 파라미터로 넘기면, 그에 해당하는 오류 메시지를 브라우저에 출력할 수 있다.
- rejectValue() : FieldError 를 검증하는 메서드.
- reject() : ObjectError 를 검증하는 메서드.

```java
/* 검증 로직 예제 */

if (!StringUtils.hasText(item.getItemName())) {
   //ItemName 이 공백이라면,
   bindingResult.rejectValue("itemName", "required");
   //itemName 필드의 required 에러 코드를 bindingResult에 담는다.
}

//bindingResult 에 에러가 담겨있다면,
if (bindingResult.hasErrors()) {
   //사용자가 입력한 폼으로 되돌려보낸다.
   return "validation/v2/addForm";
}
```

### 3️⃣ &nbsp; errors.properties

- 오류 메시지를 구분하기 쉽게 별도의 파일로 관리하자.
- 오류 코드를 만들 때는 자세히 만들 수 있고, 단순하게 만들 수도 있다.

```properties
#level1 - 자세히 만들기 (에러명.object명.field명)
required.item.itemName=상품 이름은 필수 입니다.

#level2 - 단순하게 만들기
required=필수 값 입니다.
```

- 단순하게 만들면 범용성이 좋아서 여기저기서 사용할 수 있지만 메시지를 세밀하게 만들 수 없고, 반대로 너무 자세하게 만들면 범용성이 떨어진다.
- 스프링은 세밀하게 작성된 에러 코드(level1)를 우선으로 적용하기 때문에 상황에 맞게 작성해두면 된다.

### 4️⃣ &nbsp; 검증 오류를 적용하는 3가지 방법

```
1.  스프링이 FieldError 를 생성해서 BindingResult 에 직접 넣어준다.
2.  개발자가 직접 넣어준다.
3.  Validator 를 분리한다.
```

- 검증 요구사항 중 <1번. 타입 검증>에서 가격, 수량에 문자열을 넣으면 @ModelAttribute 를 통해 Item 객체에 데이터가 주입되지 않는다. 이러한 경우는 typeMismatch로 스프링이 자동으로 FieldError 를 생성해준다.

- 검증 요구사항 중 <3번. 특정 필드의 범위를 넘어서는 검증> 은 ObjectError 에 해당하는 글로벌 오류로, 개발자가 직접 자바 코드로 에러상황을 정의한 이후 직접 bindingResult 에 에러코드를 담는다.

```java
/* 개발자가 직접 정의하는 검증 로직 */
if (item.getPrice() != null && item.getQuantity() != null) {
    int resultPrice = item.getPrice() * item.getQuantity();
    if (resultPrice < 10000) {
        bindingResult.reject("totalPriceMin", new Object[]{1000, resultPrice}, null);
    }
}
```

- 하지만 이러한 검증 로직이 '컨트롤러'에 들어가 있는 것이 문제가 될 수 있다. 컨트롤러의 역할이 커지고 유지보수성이 떨어지기 때문에 Validator 라는 클래스를 별도로 만들어서 로직을 분리하는 것이 좋다.

```java
public String addItemV5(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    //검증 로직을 수행 (타겟, BindingResult 를 파라미터로 넘긴다)
    itemValidator.validate(item, bindingResult);

    //검증에 실패하면 다시 입력 폼으로
    if (bindingResult.hasErrors()) {
        log.info("errors = {}", bindingResult);
        return "validation/v2/addForm";
    }

    //오류가 없다면 성공 로직 수행
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v2/items/{itemId}";
}
```
