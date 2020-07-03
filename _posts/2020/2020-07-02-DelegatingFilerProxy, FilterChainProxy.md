---
layout: post
title: DelegatingFilterProxy, FilterChainProxy
tags: [스프링, Spring, 시큐리티, SpringSecurity, DelegatingFilterProxy, FilterChainProxy]
---

![filter01]({{ "/assets/img/202007/filter01.png" | relative_url }})

스프링 웹 프로젝트에서 HTTP 요청이 Servlet(`DispatcherServlet`)에게 도달하기 전에 설정된 `Filter`들을 거친다. 서블릿 컨테이너에 등록된 Filter에서는 HTTP 요청을 자신에서 끝내거나 다음 필터를 호출하며 필터들이 순서를 가지고 엮여있는 것을 `FilterChain`이라 한다.

```java
// FilterChain 사용예시
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    // 요청에 대한 전처리
    chain.doFilter(request, response); // 나머지 필터 호출
    // 요청에 대한 후처리
}
```

하지만 서블릿 컨테이너에 등록되는 필터는 여러가지 스프링의 이점(빈 주입, Lifecycle Interface 등)을 활용할 수 없다. 그래서 중간다리 역할로 사용되는 것이 `DelegatingFilterProxy`이다. `DelegatingFilterProxy`는 스프링의 `ApplicationContext`에서 `springSecurityFilterChain` 이름의 빈(`FilterChainProxy`)을 찾아 실행을 위임한다. 

```java
// DelegatingFilterProxy의 실행위임 간략코드
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    // 설정된 이름의 스프링 컨테이너에 등록된 빈을 불러와서 실행을 위임한다.
    Filter delegate = getFilterBean(someBeanName);
    delegate.doFilter(request, response);
}
```

`DelegatingFilterProxy`에 의해 위임되어 실행되는 `FilterChainProxy`는 `SecurityFilterChain` 리스트를 프로퍼티로 가지고 있다. SecurityFilterChain는 다시 여러개의 Security Filter가 엮인 형태로 실제 보안과 관련된 활동이 수행되는 클래스들이다. 만일 Security Filter에 대한 동작이 궁금하다면 디버그의 시작점으로 FilterChainProxy를 활용하자.
