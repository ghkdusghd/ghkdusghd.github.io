---
title: FrontController 기본구조
parent: Spring MVC
nav_order: -1
---

# 00. 기본구조 (회원 관리 form)

> 앞으로의 MVC 학습을 위한 예제로 사용하기 위해 아래와 같이 회원관리 폼을 만들었다. 앞으로 이 코드들을 리팩토링 하며 좋은 아키텍처 설계 방법을 배우고 Spring MVC 프레임워크가 등장한 배경 및 작동 원리를 근본적인 부분부터 파악하며 학습하고자 한다.

### 1️⃣ &nbsp; MemberFormServlet

- 클라이언트를 메인 화면인 new-form.jsp 로 전송해주는 역할을 한다.

```java

@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}

```

### 2️⃣ &nbsp; MemberSaveServlet

- 클라이언트가 new-form.jsp 에서 입력한 값(username, age)을 파라미터로 받아와서 메모리 저장소에 저장하는 역할을 한다.

```java

@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")
public class MvcMemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        // Model 역할로 request 객체 내부의 저장소를 이용한다.
        request.setAttribute("member", member);

        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);

    }
}
```

### 3️⃣ &nbsp; MemberListServlet

- 메모리 저장소에 저장된 Member 객체를 끌어와서 request 객체에 저장한 후 브라우저에 출력하는 역할을 한다.

```java

@WebServlet(name = "mvcMemberListServlet", urlPatterns = "/servlet-mvc/members")
public class MvcMemberListServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        List<Member> members = memberRepository.findAll();

        request.setAttribute("members", members);

        String viewPath = "/ERB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request,response);
    }
}
```
