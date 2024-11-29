---
title: "[Spring Security + JWT] Spring Boot 로그인, 회원가입 구현"
# excerpt: "NCBT 프로젝트 시작!"

writer: Hwayeon Hong
# categories:
#   - projects
tags:
  - [spring security, jwt]

toc: true
toc_sticky: true

date: 2024-10-29
last_modified_at: 2024-10-29
---

> 개인적으로 ncbt 프로젝트 이전에 두 개의 개발 프로젝트에서 풀스택으로 개발을 해봤는데, 웹 소켓 구현만 담당했던 터라 스프링 시큐리티, JWT 토큰 구현을 정말 해보고 싶었다. 그 부분을 팀원에게 어필하여 이번 프로젝트에서 담당하게 되었다. 구현하면서 공부했던 내용들을 정리해보고자 한다.

### 1️⃣ 스프링 시큐리티 + JWT 토큰 생성 과정

내가 구현한 코드를 정리하면서 시큐리티의 동작 과정을 이해해보려고 한다. 일단 설정 파일부터 보자면, 나는 JWT 토큰을 사용할 것이기 때문에 SecurityFilterChain 에서 csrf, 세션을 무효화 했다. 그리고 토큰 발급을 위한 회원가입, 로그인 경로는 인증이 필요 없도록 열어두고, 나머지 경로는 인증이 필요하도록 설정했다.

🔥 CSRF (Cross-Site Request Forgery) 무효화

- csrf 란 보안 공격의 일종으로, 사용자가 인증된 세션을 공격자가 탈취하여 원치 않는 작업을 수행하게 하는 공격이다. JWT 는 인증 정보를 클라이언트 측에서 담고 있으며, 일반적으로 HTTP 헤더에 담아 전송한다. 때문에 서버에서는 JWT 가 유효한지 검증하기만 하면 된다. 따라서 csrf 보호 기능을 사용할 필요가 없어서 비활성화 했다.

🔥 세션 무효화

- JWT 는 stateless 한 방식으로, 세션을 사용하지 않아도 인증 및 권한 확인이 가능하기 때문에 서버에서는 세션 상태를 저장할 필요가 없다. 서버 자원 소비를 줄이기 위해 비활성화 했다.

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.NEVER))

            .authorizeHttpRequests(
                    authorize -> authorize
                            .requestMatchers("/error").permitAll()
                            .requestMatchers("/form/**").permitAll() // 일반로그인, 회원가입 요청 허용
                            .requestMatchers("/oauth/**").permitAll() // 소셜로그인, 회원가입 요청 허용
                            .requestMatchers("/admin/**").hasAuthority("ROLE_ADMIN") // ADMIN 권한이 있어야 요청할 수 있는 경로
                            .anyRequest().authenticated() // 그 밖의 요청은 인증 필요
            )

            /**
             * UsernamePasswordAuthenticationFilter 전에 JwtAuthFilter 를 거치도록 한다.
             * 토큰이 유효한지 검사하고 유효하다면 인증 객체 (Authentication) 를 가져와서 Security Context 에 저장한다.
             * (요청을 처리하는 동안 인증 정보를 유지하기 위함)
             */
            .addFilterBefore(new JwtAuthFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class);

    return http.build();
}
```

### [로그인 요청]

사용자가 form 을 통해 로그인 정보가 담긴 요청을 보낸다. 컨트롤러에서 UserService 의 generateJwtToken() 메서드를 호출하여 토큰 생성 단계로 넘어간다.

```java
@PostMapping("/form/login")
public JwtToken login(@RequestBody LoginDTO loginDTO) {
    String username = loginDTO.getUsername();
    String password = loginDTO.getPassword();
    JwtToken jwtToken = userService.generateJwtToken(username, password);
    return jwtToken;
}
```

### [토큰 생성]

스프링의 AuthenticationFilter 가 해당 요청을 받아서 UsernamePasswordAuthenticationToken (인증용 객체) 를 생성한다. 그리고 Authentication Manager 에게 처리를 위임한다. Manager 는 Token 을 처리할 수 있는 Provider 를 리스트 형태로 갖고 있는데, 그 중 적절한 Provider 를 선택하여 인증 절차를 시작한다.

나는 커스텀한 UserDetailsService 를 사용할 것이기 때문에 스프링에서 이를 인식할 수 있도록 config 파일에 이 부분을 추가해주었다.

```java
public JwtToken generateJwtToken(String username, String password) {

    // username + password 기반으로 Authentication 객체 생성
    UsernamePasswordAuthenticationToken authenticationToken =
            new UsernamePasswordAuthenticationToken(username, password);

    // 해당 user 에 대한 검증 진행
    // 이 과정에서 CustomUserDetailsService 에서 만든 loadUserByUsername 메서드가 실행된다..
    Authentication authentication = authenticationManager.authenticate(authenticationToken);

    // 인증되면 JWT 토큰 생성
    JwtToken jwtToken = jwtTokenProvider.generateToken(authentication);

    return jwtToken;
}
```

```java
@Bean
public AuthenticationManager authenticationManager(HttpSecurity http) throws Exception {
    AuthenticationManagerBuilder authenticationManagerBuilder =
            http.getSharedObject(AuthenticationManagerBuilder.class);
    authenticationManagerBuilder.userDetailsService(customUserDetailsService).passwordEncoder(passwordEncoder());
    return authenticationManagerBuilder.build();
}
```

### [인증 절차]

인증 절차가 시작되면 Authentication Provider 인터페이스가 실행되고 DB 에 저장된 사용자 정보를 가져오기 위해 UserDetailsService 인터페이스가 실행된다. UserDetailsService 는 loadUserByUsername() 메서드를 호출하여 DB에 있는 사용자의 정보를 UserDetails 형으로 가져온다.

이렇게 가져온 사용자 정보와 form 에서 넘어온 정보를 비교하고, 일치하면 Authentication 객체를 만들고, 일치 하지 않으면 예외를 던진다.

인증 완료된 사용자 정보를 가진 Authentication 객체는 SecurityContextHolder 에 담아 AuthenticationSuccessHandler 를 실행한다. 이렇게 하면 요청을 처리하는 동안에 인증 정보가 유지된다.

```java
@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    kr.kh.backend.domain.User user = userMapper.findByUsername(username);

    if(user != null) {
        // 해당하는 User 데이터가 있다면 UserDetails 객체로 만든다.
        return createUserDetails(user);
    } else {
        throw new UsernameNotFoundException("해당하는 회원을 찾을 수 없습니다.");
    }
}

private UserDetails createUserDetails(kr.kh.backend.domain.User user) {
    log.info("createUserDetails = {}", user);
    return User.builder()
            .username(user.getUsername())
            .password(user.getPassword())
            .roles(user.getRoles())
            .build();
}
```

### [토큰 생성]

Authentication 객체에서 토큰에 담을 권한 정보를 가져온다. 이 부분이 추후 토큰 인증 부분에서 문제가 되었는데, (트러블슈팅은 따로 정리할 예정이다) 권한 정보를 문자열 형태로 바꿔서 토큰에 넣어주었다.

인증 방식은 Bearer로, claims 에는 닉네임과 권한이 들어가게 만들었다. 액세스 토큰은 1시간 지속되고, 리프레쉬 토큰은 1주일 지속되게 설정했다. 아직 액세스 토큰을 어디에 저장할지 상의중에 있지만 일단 세션에 저장해두기로 했고, 이 부분은 추후 회의를 해서 정하기로 했다.

```java
public JwtToken generateToken(Authentication authentication) {
    // 유저 권한 가져오기
    String authorities = authentication.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.joining(","));

    long now = System.currentTimeMillis();

        // access token 생성 : 인증된 사용자의 권한 정보와 만료 시간을 담는다. (1시간)
        Date expiration = new Date(now + 1000 * 60 * 60);
    String accessToken = Jwts.builder()
            .setSubject(authentication.getName())
            .claim("auth", authorities)
            .setExpiration(expiration)
            .signWith(key, SignatureAlgorithm.HS256)
            .compact();

        // refresh token 생성 : access token 의 갱신을 위해 사용된다. (1주일)
        String refreshToken = Jwts.builder()
                .setExpiration(new Date(now + 1000 * 60 * 60 * 24 * 7))
            .signWith(key, SignatureAlgorithm.HS256)
            .compact();

    return JwtToken.builder()
            .grantType("Bearer")
            .accessToken(accessToken)
            .refreshToken(refreshToken)
            .build();
}
```

<img width="1016" alt="스크린샷 2024-10-06 오후 2 20 14" src="https://github.com/user-attachments/assets/99675574-381b-42a6-9e21-3b9dd76742dd">

### 2️⃣ JWT 토큰 인증 과정

테스트를 위해 SecurityFilterChain 에서 "/admin/\*\*" 경로의 요청은 "ROLE_ADMIN" 권한이 있어야만 접근 가능하도록 설정해 주었다. 해당 경로로 요청이 들어오면 JwtAuthFilter 에서 사용자 권한을 확인할 것이다.

```java
@PostMapping("/admin/test")
public String loginTest() {
    log.info("admin test controller");
    return "login success";
}
```

```java
.authorizeHttpRequests(
        authorize -> authorize
                .requestMatchers("/error").permitAll()
                .requestMatchers("/form/**").permitAll() // 일반로그인, 회원가입 요청 허용
                .requestMatchers("/oauth/**").permitAll() // 소셜로그인, 회원가입 요청 허용
                .requestMatchers("/admin/**").hasAuthority("ROLE_ADMIN") // ADMIN 권한이 있어야 요청할 수 있는 경로
                .anyRequest().authenticated() // 그 밖의 요청은 인증 필요

.addFilterBefore(new JwtAuthFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class);
)
```

### [사용자 권한 확인]

클라이언트에서 HTTP 인증 헤더로 전송한 토큰을 추출하고, 추출한 토큰은 validateToken() 메서드로 유효성 검사를 진행한다.

우선 사용자가 전송한 토큰이 우리 서버에서 발급한 정상적인 토큰이 맞는지 확인하기 위해 secret key 로 생성한 서명이 일치하는지 확인한다. 이후에 토큰이 헤더, 페이로드, 서명 형식으로 스프링에서 인식할 수 있는 방식으로 작성되었는지 확인하고 기간 만료 혹은 claims 안의 데이터가 비어있는지 여부를 확인한 후, 정상 토큰임이 확인되면 true 를 반환한다.

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    log.info("do JWT Filter ! request = {}", request);

    HttpServletRequest httpRequest = (HttpServletRequest) request;

    // 로그인 요청을 예외 처리
    if ("/form/login".equals(httpRequest.getRequestURI())) {
        log.info("httpUri = {}", httpRequest.getRequestURI());
        chain.doFilter(request, response);
        return;
    }

    // Request Header 에서 JWT 토큰 추출
    String token = resolveToken((HttpServletRequest) request);
    log.info("token is null ? {}", token);

    // validate Token 으로 유효성 검사
    if(token != null && jwtTokenProvider.validateToken(token)) {
        log.info("validate token");
        // 토큰이 유효할 경우 토큰에서 Authentication 객체를 가져와서 SecurityContext 에 저장
        Authentication authentication = jwtTokenProvider.getAuthentication(token);
        SecurityContextHolder.getContext().setAuthentication(authentication);
        log.info("set Authentication to SecurityContextHolder");
    }

    log.info("success JWT Filter ! SecurityContextHolder = {}", SecurityContextHolder.getContext());
    chain.doFilter(request, response);
}

/**
 * 토큰 추출 : 주어진 HttpServletRequest 의 Authorization 헤더에서 Bearer 접두사로 시작하는 토큰을 추출하여 반환
 */
private String resolveToken(HttpServletRequest request) {
    log.info("resolveToken request = {}", request);
    String bearerToken = request.getHeader("Authorization");
    if(bearerToken != null && bearerToken.startsWith("Bearer")) {
        log.info("bearerToken = {}", bearerToken);
        return bearerToken.substring(7);
    }
    log.info("Token null");
    return null;
}

/**
 * 토큰 유효성 검사
 */
public boolean validateToken(String accessToken) {
    log.info("do validate Token ! access token: {}", accessToken);
    try {
        Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(accessToken);
        return true;
    } catch (SecurityException | MalformedJwtException e) {
        log.info("Invalid JWT token, {}", e);
    } catch (ExpiredJwtException e) {
        log.info("Expired JWT token, {}", e);
    } catch (UnsupportedJwtException e) {
        log.info("Unsupported JWT token, {}", e);
    } catch (IllegalArgumentException e) {
        log.info("JWT claims string is empty, {}", e);
    }
    return false;
}
```

### [인증 완료]

토큰이 유효함이 확인되면, 인증 정보를 Authentication 객체에 저장하여 다음 필터로 넘긴다. 내가 구현한 코드에서는 컨트롤러로 넘어가서 "login success" 라는 문자열이 출력되게 만들었다.

<img width="1021" alt="스크린샷 2024-10-06 오후 2 21 17" src="https://github.com/user-attachments/assets/12a8cd0a-04c4-4555-b9d7-5224cc97d9f1">

이렇게 일반 로그인 + JWT 구현은 완료되었다. 프론트와의 통합은 팀원이 담당해주었고, 나는 oauth 로그인까지 마저 구현하면 될 듯 하다. 이론으로만 볼 때는 복잡했던 내용이었는데 직접 구현해보니 어느 정도는 알 것 같다는 느낌이 든다. 프로젝트가 마무리되면 지금의 미흡한 코드들을 리팩토링 해보고 싶다 ㅠㅠ

🔖 참고자료

[Spring Security + JWT 로 로그인 구현하기](https://suddiyo.tistory.com/entry/Spring-Spring-Security-JWT-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-2)
