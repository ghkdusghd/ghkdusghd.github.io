---
title: 필터와 인터셉터
parent: Spring MVC
nav_order: -16
---

# 14. 필터, 인터셉터

> 어플리케이션의 여러 로직에서 공통으로 처리해야할 로직이 있다면 필터나 인터셉터를 사용해서 처리하면 좋다. 이러한 공통 관심사는 스프링의 AOP로도 해결할 수 있지만, 웹과 관련된 공통 관심사는 HTTP 헤더나 URL 정보가 필요하다. 서블릿에서는 '필터'를, 스프링에서는 '인터셉터' 라는 기능을 지원한다.

### 1️⃣ &nbsp; Filter

```HTML
⭕️ 필터의 흐름
HTTP 요청 > WAS > 필터 > 서블릿 > 컨트롤러
```

- 필터에서 적절하지 않은 요청이라고 판단하면 그 다음 단계로 보내지 않고 종료할 수 있다.

```HTML
⭕️ 필터 체인
HTTP 요청 > WAS > 필터1 > 필터2 > 필터3 > 서블릿 > 컨트롤러
```

- 필터는 체인으로 구성되며, 중간에 필터를 자유롭게 추가할 수 있다.

#### 1️⃣ - 1. &nbsp; 필터 구현

- 필터 인터페이스를 구현하고 등록하면 서블릿 컨테이너가 필터를 싱글톤 객체로 생성하고 관리한다.

| 메서드     | 설명                                                                        |
| ---------- | --------------------------------------------------------------------------- |
| init()     | 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출한다.                   |
| doFilter() | 고객의 요청이 올 때 마다 해당 메서드가 호출된다. 공통 로직을 구현하면 된다. |
| destroy()  | 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.                     |

- <예제>

```java
@Slf4j
public class LoginCheckFilter implements Filter {

    //필터를 통과하는 경로는 따로 설정
    private static final String[] whitelist = {"/", "/member/add", "/login", "logout", "/css/*"};

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        HttpServletResponse httpResponse = (HttpServletResponse) response;

        try {
            log.info("인증 체크 필터 시작 {}", requestURI);

            if(isLoginCheckPath(requestURI)) {
                log.info("인증 체크 로직 실행 {}", requestURI);
                HttpSession session = httpRequest.getSession(false);
                if(session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) { //로그인 안된 사용자
                    log.info("미인증 사용자 요청 {}", requestURI);
                    //로그인 페이지로 redirect
                    httpResponse.sendRedirect("/login?redirectURL=" + requestURI);
                    return;
                }
            }

            filterChain.doFilter(request, response);

        } catch (Exception e) {
            throw e;
        } finally {
            log.info("인증 체크 필터 종료 {}", requestURI);
        }
    }

    /**
     * 화이트리스트의 경우 인증 체크를 하지 않도록 설정
     */
    private boolean isLoginCheckPath(String requestURI) {
        return !PatternMatchUtils.simpleMatch(whitelist, requestURI);
    }

}
```

#### 1️⃣ - 2. &nbsp; 필터 등록

- 필터를 등록하는 방법은 여러가지 있지만, 스프링 부트를 사용한다면 FilterRegistrationBean 을 사용해서 등록하면 된다.

| 메서드                     | 설명                                                                     |
| -------------------------- | ------------------------------------------------------------------------ |
| setFilter(new LogFilter()) | 등록할 필터를 지정한다.                                                  |
| setOrder(1)                | 필터는 체인으로 동작하기 때문에 순서가 필요하다. 낮을수록 먼저 동작한다. |
| addUrlPatterns("/\*")      | 필터를 적용할 URL 패턴을 지정한다. (/\* : 모든 경로)                     |

```java
@Configuration
public class WebConfig{
//    @Bean
   public FilterRegistrationBean loginCheckFilter() {
       FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
       filterRegistrationBean.setFilter(new LoginCheckFilter());
       filterRegistrationBean.setOrder(2);
       filterRegistrationBean.addUrlPatterns("/*");

       return filterRegistrationBean;
   }
}
```

### 2️⃣ &nbsp; Interceptor

```
⭕️ 인터셉터 흐름
HTTP 요청 > WAS > 필터 > 서블릿 > 스프링 인터셉터 > 컨트롤러
```

- 스프링 인터셉터는 스프링에서 제공하는 기능이기 때문에 서블릿 실행후, 서블릿과 컨트롤러 사이에서 동작한다.

```
⭕️ 인터셉터 체인
HTTP 요청 > WAS > 필터 > 서블릿 > 인터셉터1 > 인터셉터2 > 컨트롤러
```

#### 2️⃣ - 1. &nbsp; 인터셉터 구현

- 필터와 달리 인터셉터는 컨트롤러 호출 전(preHandle), 호출 후(postHandle), 요청완료 이후(afterCompletion) 세 단계로 세분화되어있다.

| 메서드          | 설명                                                                                                                                                          |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| preHandle       | 컨트롤러 호출 전에 호출된다. (더 정확히는 핸들러 어댑터 이전에 호출된다.) preHandle 의 return 값이 true이면 다음으로 진행하고, false 면 더는 진행하지 않는다. |
| postHandle      | 컨트롤러 호출 후에 호출된다. (더 정확히는 핸들러 어댑터 호출 후에 호출된다.)                                                                                  |
| afterCompletion | 뷰가 렌더링된 이후에 호출된다. 컨트롤러에서 예외가 발생되도 항상 호출되어 예외정보를 포함해서 호출할 수 있다.                                                 |

 <img src="/assets/images/pages/cs/mvc/09. Interceptor.png">
<span style="color: #808080">출처: 김영한의 스프링MVC (인프런)</span>

```java
@Slf4j
public class LogInterceptor implements HandlerInterceptor {

    public static final String LOG_ID = "logId";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();

        //afterCompletion에서 같은 요청에 대해 같은 uuid를 쓰고싶기 때문에 request객체에 담아서 보낸다.
        request.setAttribute(LOG_ID, uuid);

        //@RequestMapping 으로 처리하는 핸들러는 'HandlerMethod'에 속한다.
        //정적 리소스를 사용하는 경우에는 ResourceHttpHandler 가 사용된다.
        if(handler instanceof HandlerMethod) {
            HandlerMethod hm = (HandlerMethod) handler;
        }

        log.info("REQUEST [{}][{}][{}]", uuid, requestURI,handler);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle [{}]", modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        String requestURI = request.getRequestURI();
        Object uuid = (String) request.getAttribute(LOG_ID);

        log.info("REQUEST [{}][{}][{}]", uuid, requestURI,handler);

        //afterCompletion 은 예외가 발생해도 무조건 호출된다.
        if(ex != null) {
            log.error("afterCompletion error!!", ex);
        }
    }
}

```

#### 2️⃣ - 2. &nbsp; 인터셉터 등록

| 메서드                                        | 설명                                                       |
| --------------------------------------------- | ---------------------------------------------------------- |
| registry.addInterceptor(new LogInterceptor()) | 인터셉터를 등록한다.                                       |
| order(1)                                      | 인터셉터의 호출 순서를 지정한다. 낮을수록 먼저 호출된다.   |
| addPathPatterns("/\*\*")                      | 인터셉터를 적용할 URL 패턴을 지정한다. (/\*\* : 모든 경로) |
| excludePathPatterns()                         | 인터셉터에서 제외할 패턴을 지정한다.                       |

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**") //모든 경로를 허용
                .excludePathPatterns("/css/**", "/*.ico", "/error"); //하지만 얘네들은 인터셉터에서 제외할거야

        registry.addInterceptor(new LoginCheckInterceptor())
                .order(2)
                .addPathPatterns("/**")
                .excludePathPatterns("/", "/members/add", "/login", "/logout", "/css/**", "/*.ico", "/error");
    }

}
```
