---
layout: post
title: SpringBoot 외부설정
tags: [Java, Spring, SpringBoot, 외부설정, application.properties, application.yml, @ConfigurationProperties, @EnableConfigurationProperties, @ConfigurationPropertiesScan]
---
## Property Binding
스프링부트의 외부설정(Property)은 다음과 같이 소스코드에서 사용할 수 있다. 

- @Value : 소스코드에서 바로 프로퍼티에 주입받아 사용
- Environment : 외부설정이 바인딩 된 `Environment`를 주입받아 사용
- @ConfigurationProperties : 외부설정이 바인딩 될 Bean(객체)을 생성하여 사용

가장 빈번하게 사용하는 방식은 `@Value("${property}")` 방식일 듯 싶지만 불러올 프로퍼티의 갯수가 많거나, 계층형의 구조를 가지고 있는 경우 등에는 적절하지 않다. 또한 프로퍼티명이나 타입을 직접 입력해야 하므로 오류에 대한 여지도 있다.
```java
class MyBean {
        @Value("${myProp}")
    private String myProp; // @Value에 정의된 프로퍼티가 바인딩된다.
}
```

스프링부트에서는 Type-safe하게 프로퍼티를 빈(bean)에 맵핑하여 사용할 수 있도록 기능을 제공하고 있다. 몇 가지 애노테이션을 이용해 프로퍼티가 바인딩된 빈을 등록하여 사용할 수 있다.  
일단 다음과 같이 YAML파일에 프로퍼티를 정의했다고 하자.

- application.yml  
프로퍼티 설정파일을 작성한다.  
일반적으로 프로퍼티명은 `is-main-tester` 처럼 소문자와 대쉬(-)를 구분자로 하는 명칭이 권장된다.  
프로퍼티명 앞의 대쉬(-)는 리스트형을 나타낼때 사용한다.


- @ConfigurationProperties(prefix = "test")  
설정파일로 정의한 프로퍼티가 맵핑될 클래스를 작성하고 `@ConfigurationProperties`을 마킹한다. 이 클래스는 프로퍼티값이 맵핑될 클래스라는 의미로 애노테이션에는 프로퍼티명의 prefix를 지정할 수 있다.  
프로퍼티 파일에서 리스트형으로 작성한 `sub-testers`를 맵핑할 수 있도록 `List<Subster> subTesters` 로 선언한 것을 볼 수 있고,  
작성된 위의 프로퍼티 파일은 kebab case(`-` 가 구분자)로 작성하였으나 클래스 변수명은 camel case이다.  
스프링부트에서는 프로퍼티명의 camel case, kebab case, underscore notation 간의 변환을 모두 지원한다.  
subTesters - sub-testers  - sub_testers
```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

// 읽어들인 프로퍼티 중 해당 prefix의 프로퍼티값을 바인딩한다.
@ConfigurationProperties(prefix = "test")
public class TesterProperties {
    private String name;
    private int cnt;
    private boolean isMainTester;
    private List<SubTester> subTesters = new ArrayList<>();

      // Getter, Setter, ToString 생략
    
        static class SubTester {
        private String name;
        private int cnt;
        private boolean isMainTester;
            
                // Getter, Setter, ToString 생략
        }
}
```

- @EnableConfigurationProperties(TesterProperties.class)  
바인딩이 완료된 TesterProperties 를 빈으로 등록하여 사용하기 위해 `@EnableConfigurationProperties`를 마킹한다.  
주로 프로퍼티 값을 사용할 `@Configuration` 파일에 마킹하며, `@ConfigurationProperties`가 마킹된 클래스를 빈으로 등록해주는 역할이다.  
만일 프로퍼티 클래스가 다수라면 `@ConfigurationPropertiesScan({"base.package1", "base.package2"})` 으로 여러 패키지를 스캔하여 등록할 수도 있다.   
프로퍼티가 제대로 바인딩 됬는지 테스를 위해 `ApplicationRunner`를 작성 후 콘솔로그를 찍어본다.
```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@EnableConfigurationProperties(TesterProperties.class)
public class PropertyRunner implements ApplicationRunner {
    private final TesterProperties testerProperties;

    public PropertyRunner(TesterProperties testerProperties) {
        this.testerProperties = testerProperties;
    }

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("=================================");
        System.out.println("TesterProperties : " + testerProperties);
        System.out.println("=================================");
    }
}
```
```text
=================================
TesterProperties : TesterProperties{name='tester', cnt=99, isMainTester=true, 
subTesters=[SubTester{name='sub-tester1', cnt=11, isMainTester=false}, 
            SubTester{name='sub-tester2', cnt=22, isMainTester=false}]}
=================================
```

## @ConfigurationProperties Validation
프로퍼티값을 바인딩 할 때 검증을 추가할 수도 있다.  
스프링의 `@Validated` 와 JSR-303에 정의된 `javax.validation` 애노테이션을 사용하면 되며 간단한 예시만 남겨본다.
```java
@ConfigurationProperties(prefix = "test")
@Validated // 검증대상임을 표시
public class TesterProperties {
    @NotNull
    private String name;
    @Max(90)
    private int cnt;
    private boolean isMainTester;
        @Valid // 내부 프로퍼티에 검증이 적용되려면 @Valid를 마킹한다.
    private List<SubTester> subTesters = new ArrayList<>();

        static class SubTester {
        @NotNull
        private String name;
        @Min(20)
        private int cnt;
        private boolean isMainTester;
        }

        // Getter, Setter, ToString 생략
}
```

---

요약하면
- 외부에서 설정된 프로퍼티 파일을 코드에서 사용하는 방법은 여러가지가 있다.
- 만일 읽어올 프로퍼티가 많거나 계층형이라면 객체에 바인딩하여 사용하는 것을 고려하자.
- 프로퍼티값을 객체에 바인딩하여 빈으로 등록 후 사용을 위해서는 `@ConfigurationProperties(prefix = "test")`, `@EnableConfigurationProperties`, `@ConfigurationPropertiesScan({"base.package1", "base.package2"})` 같은 애노테이션을 조합한다.
- 프로퍼티 바인딩시 검증을 위해서는 `@Validated`와 `javax.validation` 애노테이션을 사용한다.