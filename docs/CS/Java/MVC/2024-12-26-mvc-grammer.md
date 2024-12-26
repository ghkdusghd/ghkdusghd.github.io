---
title: 스프링 MVC 기본구조
parent: Spring MVC
nav_order: -8
---

# 07. Spring MVC - 기본구조 정리

<img src="/assets/images/pages/cs/mvc/06. SpringMVC.png">
<span style="color: #808080">출처: 김영한의 스프링MVC (인프런)</span>

### ⚡️ &nbsp; 1. &nbsp; 동작 순서

1. DispatcherServlet 은 HttpServlet을 상속받아 서블릿으로 동작한다. 스프링에서는 이 서블릿을 자동으로 등록하면서 모든 경로(urlPatterns = "/") 에 대해서 매핑하도록 해두었다. 그래서 클라이언트로부터 HTTP 요청이 들어오면 해당 서블릿이 실행되도록 만든 것이다.

2. 자 이제, 클라이언트로부터 HTTP 요청(URL)이 들어오면 DispatcherServlet은 해당 요청(URL)을 처리할 수 있는 핸들러를 조회한다. 다시 말해 해당 URL로 매핑되어있는 컨트롤러가 있는지 찾는것이다.

3. URL에 매핑된 핸들러(컨트롤러)가 있다면, 그 핸들러를 처리할 수 있는 핸들러 어댑터를 조회한다. 어댑터가 필요한 이유는, 앞선 FrontController 예제에서 봤듯이 각 컨트롤러를 서블릿에서 실행할 수 있도록 구조를 변환해주는 작업이 필요하기 때문이다.

4. 컨트롤러가 실행이 되면 리턴값으로 Model, View 가 서블릿으로 들어온다. 그러면 서블릿은 viewResolver 로 view 의 논리적 경로를 물리적 경로로 변환해주고, HTML 형식으로 클라이언트에게 응답한다.

### ⚡️ &nbsp; 2. &nbsp; 프레임워크에서 수행하는 작업

- 그림의 'DispatcherServlet', '핸들러 매핑', '핸들러 어댑터 목록 조회', '핸들러 어댑터 호출' 과정을 스프링에서는 프레임워크가 자동으로 수행한다.

### ⚡️ &nbsp; 3. &nbsp; 개발자가 수행하는 작업

- 개발자는 핸들러(컨트롤러) 를 빈으로 등록해두고, 각 컨트롤러에 논리경로를 달아준 후 viewResolver 작업을 위한 물리경로를 applcation.properties 파일에 등록해두면 된다.

```java

@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping("/new-form")
    public String newForm() {
        return "new-form";
    }

    @PostMapping("/save")
    public String save(@RequestParam("username") String username,
                             @RequestParam("age") int age,
                             Model model
    ){
        Member member = new Member(username, age);
        memberRepository.save(member);

        model.addAttribute("member", member);
        return "save-result";
    }

    @GetMapping
    public String members(Model model) {
        List<Member> members = memberRepository.findAll();

        model.addAttribute("members",members);
        return "members";
    }

}

```

이렇게 간결한 코드가 완성되었다 ! 지금부터는 스프링 프레임워크에서 제공하는 기능들을 자세히 학습해보자 !
