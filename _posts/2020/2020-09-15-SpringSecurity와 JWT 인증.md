---
layout: post
title: SpringSecurity와 JWT 인증
tags: [SpringSecurity, JWT]
---

## 스프링시큐리티 + JWT 인증

스프링시큐리티를 이용하여 필터기반의 보안설정시 로그인 성공 후 기본 전략은 세션을 생성하여 저장하는 것이다. 하지만 JWT 기반에서는 서버에 유저의 상태정보(세션)이 저장되지 않고 JWT 안에 기록된다. 일단, 세션을 저장하지 않기 위해 다음 설정을 한다.

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // ... 생략
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }
}
```

이후 로그인 및 자원요청시에는 다음의 흐름으로 진행한다.

### 로그인시

1. 로그인URL(/login)으로 요청
2. 로그인 요청은 모든 사용자에게 허용(permitAll)으로 설정하여 별도의 인증절차 없이 로그인 Controller로 처리된다. 
3. 로그인 Controller에서 ID/PW 체크하여 성공시 JsonWebToken을 생성하여 반환한다.
4. 클라이언트에 반환된 JsonWebToken은 클라이언트가 저장하고, 이후 자원요청에서 Request의 `X-AUTH-TOKEN`(임의로 정한 헤더)헤더에 삽입하여 요청한다. 

### 자원요청시

1. 자원에 해당하는 URL로 요청
2. 커스텀JWT권한확인필터(`JwtAuthenticationFilter`)에서 Request Header의 `X-AUTH-TOKEN` 값을 읽어 `Authentication` 객체를 생성하고 SecurityContextHolder의 SecurityContext에 저장한다. (헤더값이 없거나 유효하지 않은 토큰이면 정보가 저장되지 않는다.)
3. 권환학인필터에서 저장된 Authentication의 권한과 해당 URL에 접근가능한 권한을 비교하여 401(Unauthroized)/403(Forbidden) 에러로 튕겨내거나 Servlet으로 요청을 넘긴다.

위 내용은 아래 블로그를 참고하여 요약하였음.

[SPRING SECURITY + JWT 회원가입, 로그인 기능 구현](https://webfirewood.tistory.com/m/115?category=694472)
