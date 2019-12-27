---
layout: post
title: SpringApplication
tags: [Java, Spring, SpringBoot, SpringApplication, FailureAnalyzers, EventListener, WebApplicationType, ApplicationRunner, CommandLineRunner]
---
## SpringApplication
main() 메서드에서 시작되는 스프링 어플리케이션의 초기 설정을 제어할 수 있는 클래스이다.  
일반적으로는 main() 메서드 안에서 어플리케이션의 실행을 `SpringApplication.run()` 메서드에게 위임하거나, 
`SpringApplication` 객체를 만들어 run() 메서드를 실행할 수 있다.  
`SpringApplication`이 초기화 되면서 어플리케이션이 구동될 때 발생하는 이벤트나 에러시 동작하는 FailureAnalyzer 등에 대해 알아본다.
    
```java
public static void main(String[] args) {
    SpringApplication.run(MySpringConfiguration.class, args);
}
```

```java
public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App.class);
        app.run(args);
}
```

### FailureAnalyzers
어플리케이션 실행시 오류가 발생한 경우에 동작하며, 에러에 대한 내용과 에러를 고치기 위한 Action을 출력한다.  
이미 많은 `FailureAnalyzers`가 등록되어 있고, 추가도 가능하다.

```text
***************************
APPLICATION FAILED TO START
***************************

Description:
Embedded servlet container failed to start. Port 8080 was already in use.

Action:
Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

### Application Events and Listeners
스프링에서 제공하는 `ContextRefreshedEvent` 외에도 스프링부트의 `SpringApplication`은 추가적인 어플리케이션 이벤트들을 보낸다.  
어플리케이션 기동이 시작되면 각 단계에 따라서 `ApplicationStartingEvent`를 시작으로 이벤트들이 발생하고, 발생된 이벤트들은 이벤트리스너에 전달된다.  

각 단계별로 어떤 이벤트가 발생하는지는 아래 스프링부트 공식문서를 참고하자.
[https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/spring-boot-features.html#boot-features-application-events-and-listeners](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/spring-boot-features.html#boot-features-application-events-and-listeners)

이벤트리스너가 빈으로 등록되어 있고, 리스너의 표적 이벤트가 발생하면 자동으로 처리메서드가 실행된다.   
빈으로 등록된 `ApplicationStartedEvent`리스너와 자동실행된 로그이다.  
아래 클래스가 빈으로 등록되어 있고 `ApplicationStartedEvent` 이벤트가 발생하면 자동으로 `onApplicationEvent()` 메서드가 호출된다.

```java
@Component // 빈으로 등록한다.
public class SampleListener implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent applicationStartedEvent) {
        System.out.println("ApplicationStartedEvent가 발생!");
    }
}
```

어플리케이션 기동시 마지막에 콘솔로그가 출력된다.  
(`ApplicationStartedEvent` 가 기동시 제일 마지막에 발생하기 때문이다.)

```text
2019-12-26 16:09:49.823  INFO 4212 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-12-26 16:09:49.828  INFO 4212 --- [           main] com.yang.wind.App                        : Started App in 2.1 seconds (JVM running for 3.235)
ApplicationStartedEvent가 발생!
```

하지만 `ApplicationStartingEvent`의 경우 `ApplicationContext`가 초기화 되기 전에 발생하는 이벤트기 때문에, 빈 등록으로는 리스너가 동작하지 않는다.  
이런 경우에는 `SpringApplication`에 `addListeners()` 메서드로 리스너를 추가하면 된다.  
아래 코드는 `SpringApplication.run()`으로 바로 시작하지 않고, 객체 생성을 통해 설정을 추가했다.

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App.class);
        app.addListeners(new SampleListener()); // 리스너를 코드로 추가한다.
        app.run(args);
    }
}
```
```java
// 빈등록이 필요없다.
public class SampleListener implements ApplicationListener<ApplicationStartingEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartingEvent applicationStartingEvent) {
        System.out.println("ApplicationStartingEvent 발생!");
    }
}
```

어플리케이션 기동시 처음에 콘솔로그가 출력된다.
```text
ApplicationStartingEvent 발생!
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.0.RELEASE)
....
```

### WebApplicationType
`SpringApplication`에서는 `ApplicationContext`의 타입이 내부적으로 확정된다.  
3개의 타입이 존재하는데 다음과 같이 자동설정되며 코드에서 특정 타입으로 오버라이드가 가능하다. 

- WebApplicationType.SERVLET : Spring MVC가 존재하면 설정됨
- WebApplicationType.REACTIVE : Spring MVC가 없고, Spring WebFlux가 존재하면 설정됨
- WebApplicationType.NONE : 위의 경우에 해당이 안되면 설정됨. 어플리케이션이 기동 되었다가 바로 종료된다.
```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App.class);
        // ApplicationType 설정을 오버라이드 한다.
        app.setWebApplicationType(WebApplicationType.SERVLET);
        app.run(args);
    }
}
```

### Accessing Application Arguments
어플리케이션 기동시 설정한 아규먼트에 접근하고 싶다면 `org.springframework.boot.ApplicationArguments` 빈을 사용하면 된다.  
어플리케이션 기동시 커맨드나 IDE에서 옵션을 주면(JVM 옵션이 아니라 프로그램 옵션이다.) `main(String[] args)` 메서드에서 `SpringApplication`으로 전달된다. 
사용하기 원하는 클래스에 `@Autowired`나 생성자로 주입받아 사용하면 된다. `ApplicationArguments`는 `getNonOptionArgs()`, `getOptionNames()`, `getOptionValues()` 등 사용하기 편리하도록 메서드 추상화하여를 제공하고 있다.
```java
@Component
public class MyBean {
    @Autowired
    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
    }
}
```

### ApplicationRunner / CommandLineRunner
SpringApplication이 시작한 시점에 특정 코드를 실행하고자 한다면, `ApplicationRunner`나 `CommandLineRunner`를 활용할 수 있다.  
두 개의 Runner 인터페이스 중 하나를 implements하여 메서드를 오버라이드한 후 빈으로 등록해주면 사전준비는 완료된다.

- ApplicationRunner
    `run(ApplicationArguments args)` 메서드 하나면 오버라이드하면 되며 `ApplicationArguments`에 바인딩된 어플리케이션 아규먼트들에 접근이 가능하다.  
    `ApplicationArguments`는 여러가지 편리한 메서드를 제공하고 있다. 그러므로 아래에서 설명할 `CommandLineRunner`보다 사용이 권장된다.  
    아래와 같이 Option, Non-Option Argument를 설정했을 때의 테스트 코드이다.  
    `java -jar MySpringBootApp.jar alone1 alone2 --key1=value1 --key2=value2`

    ```java
    @Component
    public class SimpleApplicationRunner implements ApplicationRunner {
        @Override
        public void run(ApplicationArguments args) {
                    System.out.println("======================================================");
            System.out.println("ApplicationRunner - ApplicationArguments ");
            System.out.println("NonOption Arguments : " + args.getNonOptionArgs());
            System.out.println("Option Arguments Names : " + args.getOptionNames());
            System.out.println("key1의 value : " + args.getOptionValues("key1"));
            System.out.println("key2의 value : " + args.getOptionValues("key2"));
            System.out.println("======================================================");
        }
    }
    ```
    ```text
    ======================================================
    ApplicationRunner - ApplicationArguments 
    NonOption Arguments : [alone1, alone2]
    Option Arguments Names : [key1, key2]
    key1의 value : [value1]
    key2의 value : [value2]
    ======================================================
    ```

- CommandLineRunner
    `ApplicationRunner`와 기능은 같지만 전달되는 아규먼트가 더 불친절하다.  
    Option과 Non-Option 아규먼트의 구분도 없고 아규먼트도 단순 배열로 전달된다.  
    `java -jar MySpringBootApp.jar alone1 alone2 --key1=value1 --key2=value2`

    ```java
    @Component
    public class SimpleCommandLineRunner implements CommandLineRunner {
        @Override
        public void run(String... args) throws Exception {
            System.out.println("======================================================");
            System.out.println("CommandLineRunner - String... ");
            for(String s : args){
                System.out.println("argument : " + s);
            }
            System.out.println("======================================================");
        }
    }
    ```
    ```text
    ======================================================
    CommandLineRunner - String... 
    argument : alone1
    argument : alone2
    argument : --key1=value1
    argument : --key2=value2
    ======================================================
    ```