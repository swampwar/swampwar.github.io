---
layout: post
title: "SpringBoot"
author: Yangs
tags: [spring]
---

# SpringBoot

## 프로젝트 구조

- @EnableAutoConfiguration (@SpringBootApplication 안에 숨어 있음)
- 빈은 사실 두 단계로 나눠서 읽힘
    - 1단계: @ComponentScan
    - 2단계: @EnableAutoConfiguration
- @ComponentScan : 해당 자바파일의 패키지를 기본패키지로 하위 패키지의 컴포넌트들을 모두 빈으로 등록한다. 개발자가 빈등록을 위해 애노테이션을 마킹한 클래스들이 빈으로 등록된다.
    - @Component
    - @Configuration @Repository @Service @Controller @RestController
- @EnableAutoConfiguration : @ComponentScan이 동작한 이후 자동설정이 동작한다.
    - spring.factories
        - spring-boot-autoconfigurer 라이브러리의 META-INF 폴더 이하에 있는 파일이다.
        - org.springframework.boot.autoconfigure.EnableAutoConfiguration 에 설정된 클래스들이 모두 자동설정의 대상이 된다.
    - @Configuration : 위 자동설정 대상 클래스들은 모두 @Cofiguration이 마킹되어 있다.
    - @ConditionalOnXxxYyyZzz : 모든 자동설정이 다 적용되는 것은 아니고 자동설정 클래스별로 조건에 따라 자동설정 되거나 무시된다. 
    예를들어 @ConditionalOnMissingBean 조건에는 특정 빈이 등록되어 있지 않은 경우에만 적용된다.

    ---

    ## 자동설정

- Xxx-Spring-Boot-Autoconfigure 모듈: 자동 설정 클래스가 포함된다.(spring.factories, AutoConfiguration class 포함)
    - [XxxProperties.java](http://xxxproperties.java) 파일에는 자동설정시 참조하는 프로퍼티에 대한 정보가 있다.
    1. @ConfigurationProperties 를 XxxProperties.java 에 마킹하고, 
    2. @EnableConfigurationProperties(XxxProperties.class)가 마킹된 자동설정파일에 값을 참조하여 빈을 생성한다.
- Xxx-Spring-Boot-Starter 모듈: 필요한 의존성 정의
- 그냥 하나로 만들고 싶을 때는? Xxx-Spring-Boot-Starter

    ---