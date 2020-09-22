---
layout: post
title: Spring Validation
tags: [Spring, Validation, 검증]
---

### Spring Validation Flow

어플리케이션 진입 → Validation → 비즈니스 로직 : @Controller → Validation → @Service

### 컨트롤러 Validation Flow

1. 비즈니스를 처리하기 위한 데이터는 컨트롤러 메서드의 파라미터에 바인딩
2. 파라미터에 @Valid 로 검증하고자 하는 객체를 마킹
3. 객체 클래스에 정의된 애노테이션에 따라 검증 수행
    - 클래스에는 빈의 검증에 사용될 메타데이터가 애노테이션으로 정의되어 있다. 예를들면 @Email, @NotEmpty 등
4. 검증오류시 ExceptionHandler 에서 처리
    - 각 컨트롤러에서 오류를 처리할 수도 있지만 @ControllerAdvice - @ExceptionHandler를 이용하여 통합적으로 처리한다.

### Spring boot에서의 의존성

Maven을 사용한다면 starter로 hibernate-validator(검증에 대한 구현체) 등 필요한 의존성을 주입받을 수 있다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 참고
- [JavaBean Validation과 Hibernate Validator 그리고 Spring Boot](https://medium.com/@SlackBeck/javabean-validation%EA%B3%BC-hibernate-validator-%EA%B7%B8%EB%A6%AC%EA%B3%A0-spring-boot-3f31aee610f5)
- [Validation 어디까지 해봤니?](https://meetup.toast.com/posts/223)
