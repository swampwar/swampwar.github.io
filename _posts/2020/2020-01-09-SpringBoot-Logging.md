---
layout: post
title: SpringBoot Logging
tags: [Java, Spring, SpringBoot, Logging, 로깅, Logback, 로그]
---

### 스프링부트의 Log 라이브러리
스프링부트는 Java Util Logging, Log4J2, Logback 이렇게 3가지에 대해 기본 설정을 제공하고 있다.
spring-boot-starter-logging에 로그관련 라이브러리들이 추가되는 것을 볼 수 있다.

![SpringBoot_Logging_0]({{ "/assets/img/202001/SpringBoot_Logging_0.png" | relative_url }})

이 중에서 스프링부트는 기본으로 `Logback`을 사용하며 자동으로 이루어지는 설정은 `org.springframework.boot.logging.DefaultLogbackConfiguration` 파일을 참고한다.   
기본설정으로 아래와 같이 시간-로그레벨-프로세스ID-구분자-쓰레드명-로거명-로그내용 포맷이 출력된다.  

![SpringBoot_Logging_1]({{ "/assets/img/202001/SpringBoot_Logging_1.png" | relative_url }})

디폴트 설정에서 변경을 원한다면 프로퍼티 설정파일에서 logging으로 시작하는 옵션을 지정하거나 별도의 로그설정파일(logback.xml 등)을 사용하여 디폴트 설정을 오버라이드 한다.

### Log Level
스프링부트의 기본설정은 INFO 레벨 이상인 로그만 보여준다. 만일 상세한 로그가 보고 싶다면 debug모드로 어플리케이션을 기동하던지 출력되는 로그레벨을 프로퍼티에서 조정한다.

- 디버그 모드로 기동
    - jar파일 실행시 `--debug` 옵션을 부여한다.
    - 프로퍼티 설정파일(`application.properties`)에 `debug=true`을 추가한다.
- 어플리케이션의 출력로그 레벨을 조정
    - `logging.level`을 prefix로 하는 프로퍼티를 추가한다.
    - `logging.level.<logger-name>=<level>`  
      `logging.level.com.yang.wind.mapper=TRACE` : mapper 패키지 이하의 로그레벨을 TRACE로 한다.  
      `logging.level.root=INFO` : root 이하 모든 패키지의 로그레벨을 INFO로 한다.

### Log Color
만약 ANSI를 지원하는 터미널을 사용한다면 프로퍼티 설정파일에 `spring.output.ansi.enabled=detect` 또는 `always` 옵션을 줘서 로그를 컬러풀하게 출력할 수 있다.

### Log 파일출력
기본적으로 아무설정이 없다면 콘솔은 출력하지만 로그파일은 작성되지 않는다. 파일로 로그를 출력하고자 한다면 프로퍼티 설정파일에 `logging.file.name` 과 `logging.file.path` 프로퍼티를 추가한다.  
name과 path 두 설정을 모두 지정할 경우에는 함께 동작하지 않는다. `logback-spring.xml` 과 같이 별도의 파일로 설정을 해줘야 할 듯 싶다.  
프로퍼티 지정에 따른 동작방식은 아래를 참고하자.

![SpringBoot_Logging_2]({{ "/assets/img/202001/SpringBoot_Logging_2.png" | relative_url }})

### Log Group
특정 로거들을 그룹핑하여 설정을 동일하게 변경이 가능하다. 예를들어 Tomcat에 관련된 로거들을 동시에 설정변경 하고자 한다면 각자의 패키지로 설정을 변경하기 보다는 그룹을 지정하여 간편하게 변경할 수 있다.

프로퍼티 설정파일에서 tomcat이라는 그룹을 생성하고 설정을 변경한다.
`logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat`
`logging.level.tomcat=TRACE`

또한 스프링부트는 미리 설정된 그룹을 제공하기 때문에 편리하게 로그레벨 설정이 가능하다.
![SpringBoot_Logging_3]({{ "/assets/img/202001/SpringBoot_Logging_3.png" | relative_url }})

### Custom Log 설정
설정한 로그시스템에 따라 아래의 파일이 로드되어 환경이 구성된다. 
![SpringBoot_Logging_4]({{ "/assets/img/202001/SpringBoot_Logging_4.png" | relative_url }})