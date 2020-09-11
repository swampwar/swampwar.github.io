---
layout: post
title: SpringSecurity 의 CORS 설정
tags: [SpringSecurity, CORS]
---

## SpringSecurity 의 CORS 설정

아래 설정파일을 참고하여 스프링시큐리티에서 CORS를 설정한다.

HttpSecurity http.cors() 설정시 CorsConfigurationSource 빈이 정의되어 있으면 설정을 오버라이드 한다.  

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
				// .. 생략
        .and()
            .cors() // 인증과 무관하게 Origin헤더가 있는 모든 요청에 대해 CORS헤더를 포함한 응답을 해준다.
				// .. 생략
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.addAllowedOrigin("*");
        configuration.addAllowedMethod("*");
        configuration.addAllowedHeader("*");
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

### 참고 :
[https://oddpoet.net/blog/2017/04/27/cors-with-spring-security/](https://oddpoet.net/blog/2017/04/27/cors-with-spring-security/)