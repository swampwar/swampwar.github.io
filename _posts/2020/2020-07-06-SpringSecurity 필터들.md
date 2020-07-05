---
layout: post
title: UsernamePasswordAuthenticationFilter, RememberMeAuthenticationFilter, AnonymousAuthenticationFilter
tags: [spring, security, UsernamePasswordAuthenticationFilter, RememberMeAuthenticationFilter, AnonymousAuthenticationFilter]
---

스프링 시큐리티를 공부하며 주요 필터에 대해 정리해본다.

## UsernamePasswordAuthenticationFilter

폼인증시 동작하는 필터로 인증이 완료되면 `Authentication` 객체를 생성하여 `SecurityContext`에 저장한다.

폼인증 활성화 : 디폴트 로그인 주소인 `/login` 으로 POST 요청이 오면 `UsernamePasswordAuthenticationFilter` 가 동작한다.

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin(); // 폼인증 활성화
    }
}
```

`UsernamePasswordAuthenticationFilter` 동작 과정

![UsernamePasswordAuthenticationFilter]({{ "/assets/img/202007/filter02.png" | relative_url }})

1. 사용자의 로그인 요청시(/login) 요청된 URL이 사전설정된 로그인URL과 동일하면 `UsernamePasswordAuthenticationFilter`가 동작한다. Request로부터 username과 password를 추출하여 `UsernamePasswordAuthenticationToken` (인증이 되지않은 Token)을 생성한다. 
2. 생성한 Token을 `AuthenticationManager`에게 전달하여 인증을 요청한다.
3. 인증이 실패하면 `SecurityContextHolder`를 비우고 `AuthenticationFailureHandler` 를 실행하는 등 후처리를 수행한다. `AuthenticationFailureHandler`는 인증이 실패했을 때의 동작을 사전에 정의할 수 있다.
4. 인증이 성공하면 `SecurityContextHolder`에 인증된 정보를 가지고 새로 생성한 `Authentication`(`UsernamePasswordAuthenticationToken`)을 저장하고 `AuthenticationSuccessHandler` 를 실행하는 등 후처리를 수행한다. 여기서 생성한 `UsernamePasswordAuthenticationToken`은 이전에 생성된 인증객체와 다르게 ID/PW를 포함하여 권한등 정보를 추가로 가지고 있다.

## RememberMeAuthenticationFilter

리멤버미 인증이 활성화 되면 로그인 성공시 세션키(JSESSIONID)와 리멤버키가 같이 쿠키로 발행된다. 이후 세션이 끊기거나 만료되거나 브라우저가 재시작을 하여도 요청에 리멤버미 쿠키값이 유효하다면(지정된 만료시간이 지나기 전) 다시 로그인이 된다.

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.rememberMe(); // 리멤버미 인증 활성화
    }
}
```

## AnonymousAuthenticationFilter

앞 필터들에서 Authentication 객체가 생성되지 않은경우 동작하며 익명사용자용 `Authentication`(`AnonymousAuthenticationToken`)을 생성하여 `SecurityContextHolder`에 저장한다. (세션에는 저장하지 않는다.)  
세션에 저장되지 않으므로 요청이 처리되고나면 사라진다.

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {

		if (SecurityContextHolder.getContext().getAuthentication() == null) {
			SecurityContextHolder.getContext().setAuthentication(
					createAuthentication((HttpServletRequest) req));
		}
		else {
    
		}

		chain.doFilter(req, res);
	}
```
