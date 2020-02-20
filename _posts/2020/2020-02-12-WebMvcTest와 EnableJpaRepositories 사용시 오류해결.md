---
layout: post
title: Springboot WebMvcTest 테스트중 오류해결(feat.EnableJpaRepositories)
tags: [Java, Spring, SpringBoot, WebMvcTest, EnableJpaRepositories]
---

## 오류발생
새로운 Controller를 테스트하기 위해 테스트 소스를 작성하고 WebMvc와 관련된 빈만 등록해 테스트 하고자 했다.  
제공되는 SliceTest 애노테이션인 @WebMvcTest를 테스트클래스에 마킹하고 실행했으나 초기화 단계에서 Exception이 발생하였다.  

`org.springframework.beans.factory.NoSuchBeanDefinitionException: No bean named 'entityManagerFactory' available`

## 원인
`@WebMvcTest`로 WebMvc와 관련된 빈만 등록된 상태에서 메인클래스에 마킹한 `@EnableJpaRepositories`이 JPA와 관련된 빈을 찾아 작업을 하려할 때 오류가 발생하였다.

## 해결
QueryLookupStrategy 설정을 위해 메인클래스에 `@EnableJpaRepositories`를 마킹했으나, 굳이 필요한 설정은 아니므로 지우고 테스트를 실행하였다.(그저 학습의 이유로 디폴트 전략을 명시적으로 설정하였다.)  
만일 필요한 옵션이라면  @EnableJpaRepositories를 통해서 하지않고 다른 방법으로 옵션을 줬어야 했을것이다.

```java
@SpringBootApplication
//@EnableJpaRepositories(queryLookupStrategy = QueryLookupStrategy.Key.CREATE_IF_NOT_FOUND)
public class App {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App.class);
        app.setWebApplicationType(WebApplicationType.SERVLET);
        app.run(args);
    }
}
```

## 참고
- [참고 링크](https://github.com/spring-projects/spring-boot/issues/6844)
  > By using @EnableJpaRepositories you are explicitly telling Spring Boot's auto-configuration to back off and that you'll handle Spring Data yourself.  
    I think what's happening is that @WebMvcTest is turning off JPA (i.e. not finding your @Entity classes) but @EnableJpaRepositories is still active so it complains that it can't find any JPA models.