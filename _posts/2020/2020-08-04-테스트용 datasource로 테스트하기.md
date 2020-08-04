---
layout: post
title: 테스트용 datasource로 테스트하기
tags: [JPA, datasource, DataJpaTest]
---

## 테스트용 datasource로 테스트하기

1. src/test/resources 이하에 [application.properties](http://application.properties) 생성하여 datasource 프로퍼티를 작성한다. → 테스트시 src/main/resources/application.properties를 오버라이드 한다.
2. 별도의 설정파일(@Configuration)을 생성하여 테스트시 참조한다.(테스트용 빈등록을 한다.)
3. datasource 빈에 프로파일을 설정(@Profile)하고 테스트시 해당 프로파일을 사용한다.

### 참고

- [https://www.baeldung.com/spring-testing-separate-data-source#using-spring-profiles](https://www.baeldung.com/spring-testing-separate-data-source#using-spring-profiles)