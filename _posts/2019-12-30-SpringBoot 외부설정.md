---
layout: post
title: SpringBoot 외부설정
tags: [Java, Spring, SpringBoot, 외부설정, application.properties, application.yml]
---
## 외부설정(Externalized Configuration)
스프링부트는 외부설정을 통해 같은 소스코드로 각기 다른 환경에서 실행될 수 있도록 해준다.  
코드의 외부설정으로는 properties 파일, YAML 파일, 환경변수, 커맨드라인 아규먼트 등으로 설정이 가능하다.

외부설정(Property)은 다음과 같이 소스코드에서 사용할 수 있다.
- @Value : 소스코드에서 바로 프로퍼티에 주입받아 사용
- Environment : 외부설정이 바인딩 된 Environment 를 주입받아 사용
- @ConfigurationProperties : 외부설정이 바인딩 될 Bean(객체)을 생성하여 사용

외부설정을 할 수 있는 방법이 여러가지가 있으므로 이들 사이에 우선순위도 존재한다.  
같은 프로퍼티를 여기저기서 설정했다면 정의된 우선순위에 따라서 가장 높은 우선순위가 나머지를 오버라이드 한다.

스프링부트 공식문서에 소개된 우선순위는 17가지이므로 우선순위에 대한 개념만 알고 넘어가며 상황에 따라 찾아보면 될 듯하다.   
기본적인 classpath 이하에 `application.properties` 파일을 두는 것은 우선순위가 15위로 하위권이다. 당연히 3순위인 `Command line argument`를 부여하면 오버라이드 된다.  
예를들어 `application.properties`에 `foo = bar1`로 작성하고 `java -jar myproject.jar --foo=bar2` 로 어플리케이션을 실행하면 어플리케이션에서 foo에 맵핑된 프로퍼티값은 'bar2'가 된다.

> 외부설정의 우선순위  
> [https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config)

### Application Property Files
여러가지 방법으로 외부설정이 가능하다 하더라도 가장 많이 사용할 외부설정 방식은 프로퍼티 파일일 것이다.  
주로 classpath:/application.properties 경로에 설정파일을 두고 사용하고 있지만 여러 경로에 두는 것이 가능하다.

프로퍼티 파일의 위치에 따라서 정의된 우선순위가 있다. 마찬가지로 같은 프로퍼티가 여러 파일에 정의되어 있다면 우선순위가 높은 경로에 있는 파일이 오버라이드한다.

공식문서에서 소개하는 우선순위이다. classpath 바로 이하에 두는것은 우선순위가 제일 낮은 6위이다. classpath 이하에 디폴트 설정파일을 두고 필요에 따라 실제 프로퍼티값을 가진 파일을 상위 우선순위 위치에 두는 전략을 생각해볼 수 있다.

1. file:./custom-config/
2. classpath:custom-config/
3. file:./config/
4. file:./
5. classpath:/config/
6. classpath:/

위 경로중에 `custom-config` 디렉터리는 개발자가 직접 설정이 가능한 경로라는 의미이다.  
`spring.config.name`와 `spring.config.location` 프로퍼티로 커스텀한 파일경로를 지정한다. 프로퍼티 파일경로에 대한 설정은 기동시 이른 시점에 결정되어야 하므로 OS환경변수나 시스템 프로퍼티 또는 커맨드라인 아규먼트로 지정해줘야 한다.  
아래는 커맨드라인 아규먼트로 지정한 예시이다. 경로(파일도 가능)는 컴마(,)를 구분자로 여러개를 지정할 수 있다.

- `spring.config.name` 프로퍼티로 파일명만 지정한다.
```shell
$ java -jar myproject.jar --spring.config.name=myproject
```
- `spring.config.location` 프로퍼티로 경로와 파일명까지 지정한다. 
```shell
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
```

---

요약하면
- 스프링부트는 코드 밖에서 외부 프로퍼티 설정이 가능하다.
- 외부는 properties 파일, YAML 파일, 환경변수, 커맨드라인 아규먼트 등을 말하며 이들사이에는 우선순위가 있어 프로퍼티값이 오버라이드 된다.
- 외부설정 중 많이 쓰이는 '파일 외부설정'은 파일이 위치하는 경로에 따라 우선순위가 있어 프로퍼티값이 오버라이드 된다.
- 파일 외부설정시 디폴트 파일명, 경로 외에 `spring.config.name`, `spring.config.location` 프로퍼티 설정으로 커스텀한 파일명 또는 경로를 지정할 수 있다.