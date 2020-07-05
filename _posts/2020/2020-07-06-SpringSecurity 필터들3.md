---
layout: post
title: ConcurrentSessionFilter, SessionManagementFilter
tags: [spring, security, SessionManagementFilter, ConcurrentSessionFilter]
---

스프링 시큐리티를 공부하며 주요 필터에 대해 정리해본다.

## ConcurrentSessionFilter, SessionManagementFilter

세션과 관련된 설정은 `HttpSecurity http.sessionManagement()` 이하 메서드로 설정하고, 설정 값을 `ConcurrentSessionFilter`와 `SessionManagementFilter`에서 읽어서 세션을 처리한다.

한 계정으로 여러 곳에서 로그인을 하는경우 동시세션이 발생하며 `maximumSessions()` 으로 동시에 발생할 수 있는 세션의 갯수를 설정한다.  
max 세션 갯수를 초과하는 요청에 대해서는 `maxSessionsPreventsLogin()` 으로 이미 생성된 세션을 만료할지 신규 세션 요청을 막을지 설정한다.

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.sessionManagement()
            .maximumSessions(1) // 계정당 생성될 세션 갯수, '-1'은 무제한 생성
            .maxSessionsPreventsLogin(true) // 동시세션 발생시 이후 세션에 대해 '이후 로그인'을 막을지 여부, false면 이전 사용자의 세션이 만료처리된다.
            .expiredUrl("/expired"); // 세션만료시 이동페이지
    }
}
```

공격자가 사용자에게 강제로 공격자의 JSESSIONID를 주입 후 로그인하게 하여 세션을 공유하는 `세션고정공격`의 경우 스프링시큐리티의 디폴트 설정으로 보호된다. `changeSessionId()`설정으로 세션 생성시 JSESSIONID가 변경된다. (사용자가 로그인하면 공격자가 심은 JSESSIONID에서 새로운 JSESSIONID로 변경되어 세선공유가 일어나지 않는다.)

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.sessionManagement()
            .sessionFixation().changeSessionId(); // 기본값인 changeSessionId() 외에 none(), migrateSession(), newSession()이 있다.
    }
}
```

스프링시큐리티의 세션생성에 관한 정책은 `sessionCreationPolicy()`으로 가능하다. 세션을 생성하지도, 사용하지도 않을 경우(JWT를 활용한 인증 등)는 `SessionCreationPolicy.STATELESS` 을 파라미터로 전달한다. 기본값은 `SessionCreationPolicy.IF_REQUIRED` 으로 스프링시큐리티가 필요시 세션을 생성하여 사용하는 옵션이다.

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.sessionManagement()
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
	}
}
```