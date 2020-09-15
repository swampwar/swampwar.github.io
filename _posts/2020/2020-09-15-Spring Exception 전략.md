---
layout: post
title: Spring Exception 전략
tags: [Spring, ExceptionHandler, ControllerAdvice]
---

## Spring Exception 전략

### @ControllerAdvice + @ExceptionHandler 로 모든 Exception을 처리

- @ControllerAdvice : 여러 컨트롤러에서 적용되는 글로벌 코드를 작성할 수 있도록 해주는 애노테이션
- @ExceptionHandler :  Exception 발생시 처리될 코드를 작성할 때 사용하는 애노테이션

2개의 애노테이션을 조합하여 어플리케이션에 발생하는 Exception을 처리하는 로직을 한 곳에 모을 수 있다.
Exception 처리로직이 여러 곳에 분산되어 발생할 수 있는 관리의 어려움이나 누락을 방지한다.

컨트롤러 → 서비스1 → 서비스2 ... 로 이어지는 코드에서 중간에 Exception이 발생하면 Controller로 throw되고 try~catch 문으로 처리하지 않으면 최종적으로 @ExceptionHandler에서 처리된다.

### ErrorResponse 객체 사용

클라이언트와 서버 사이의 약속으로 어떤 데이터가 들어있는지 명확하게 표현된다.

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class ErrorResponse {
    private String message;
    private int status;
    private List<FieldError> errors;
    private String code;
    ...

    @Getter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class FieldError {
        private String field;
        private String value;
        private String reason;
        ...
    }
}
```

### BusinessException 사용

비즈니스 로직에서 더이상 진행이 불가능한 경우(기간만료, 미사용 상태 등) 사전에 BusinessException을 정의하여 throw한다. if~else 문으로 여러 케이스를 모두 처리하는 것은 코드를 복잡하게 하고 유지보수를 어렵게 한다. 단순하게 단일책임을 부여하고 처리할 수 없는 상태는 Exception을 throw한다. 

### 참고

- [Spring Guide - Exception 전략](https://cheese10yun.github.io/spring-guide-exception/)
- [Global exception handling with @ControllerAdvice](https://lankydan.dev/2017/09/12/global-exception-handling-with-controlleradvice)
