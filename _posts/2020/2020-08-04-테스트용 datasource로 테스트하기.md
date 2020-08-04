---
layout: post
title: 테스트용 datasource로 테스트하기
tags: [JPA, datasource, DataJpaTest]
---

## 테스트용 datasource로 테스트하기

1. `src/test/resources` 이하에 `application.properties` 생성하여 datasource 프로퍼티를 작성한다. → 테스트시 `src/main/resources/application.properties`를 오버라이드 한다.
2. 별도의 설정파일(`@Configuration`)을 생성하여 테스트시 참조한다.(테스트용 빈등록을 한다.)
3. datasource 빈에 프로파일을 설정(`@Profile`)하고 테스트시 해당 프로파일을 사용한다.

## 1번 방식을 이용하려고 작성해본 테스트용 `application.yml`

```yaml
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb
    username: sa
    password:

  jpa:
    hibernate:
      ddl-auto: create # DDL적용방식 설정
    show-sql: true # SQL출력 설정
    properties:
      hibernate.format_sql: true # SQL출력 포맷팅 설정

logging:
  level:
    root: info
    org.hibernate.type.descriptor.sql: debug
```

### 참고

- [https://www.baeldung.com/spring-testing-separate-data-source#using-spring-profiles](https://www.baeldung.com/spring-testing-separate-data-source#using-spring-profiles)