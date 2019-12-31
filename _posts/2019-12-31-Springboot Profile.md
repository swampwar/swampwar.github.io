---
layout: post
title: SpringBoot Profile
tags: [Java, Spring, SpringBoot, Profile, application.properties, application.yml]
---
스프링에서는 Profile을 지정하여 어플리케이션의 구성(Configuration)을 Profile별로 가져갈 수 있도록 한다. 즉, 같은 소스코드를 두고 로컬환경, 개발환경, 운영환경별로 다른 구성(DB 접속정보, 업로드 파일의 저장위치 등)으로 실행할 수 있다.

## @Profile
다르게 가져갈 구성은 @Component, @Configuration, @ConfigurationProperties 애노테이션에 @Profile을 마킹하여 지정하고, 어플리케이션 실행시 `spring.profiles.active` 프로퍼티로 프로파일을 지정하면 해당 프로파일이 적용된 빈만 등록된다. 

아래는 `test` 프로파일을 적용해본 예시이다.
```java
@Component
@Profile("test") // 이 클래스는 [test]프로파일에서만 빈으로 등록된다.
public class TestBean {
}
```

```java
@Component
public class Sample implements ApplicationRunner {
    @Autowired(required = false)
    TestBean testBean;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("=====================================================");
        System.out.println("test 프로파일에서만 등록되는 빈 : " + testBean);
        System.out.println("=====================================================");
    }
}
```

`--spring.profiles.active=test` 옵션을 주고 실행시 다음과 같이 콘솔에 남으며 TestBean 빈이 null 값이 아니라 등록 되었음을 확인할 수 있다.

```text
=====================================================
test 프로파일에서만 등록되는 빈 : com.yang.wind.TestBean@50fb33a
=====================================================
```

만일 `--spring.profiles.active=test` 옵션을 빼고 실행하여 디폴트 프로파일로 실행하거나 `--spring.profiles.active=dev` 처럼 다른 프로파일을 지정하면 null 값으로 콘솔에 남는다.

## Profile - Property File
스프링부트는 활성화 된 프로파일에 따라서 지정된 프로퍼티 파일을 읽는다.  
즉 어떠한 프로파일도 활성화 하지 않으면 디폴트 프로퍼티 파일로 `application.properties`를 읽지만 `test` 프로파일을 활성화 하면 프로퍼티 파일로 `application-test.properties`를 읽어온다. 하지만 `application-test.properties`만 읽는것이 아니라 `application.properties` 파일도 같이 읽으며 중복된 값이 profile이 정의된 파일로 오버라이드 된다.

- XXX 프로파일 활성화시 설정파일 : application.properties, application.yml, application-XXX.properties, application-XXX.yml

프로파일에 따라 어떻게 설정파일을 읽는지 알기위해 간단한 테스트를 해본다.

```yaml
# application.yml
profile-common:
  name: defaultnameCommon

defaultonly:
  name: defualtname
```

```yaml
# application-dev.yml
profile-common:
  name: devnameCommon

devonly:
  name: devname
```

```yaml
# application-test.yml
profile-common:
  name: testnameCommon

testonly:
  name: testname

spring:
  profiles:
    include: include
```

```yaml
# application-include.yml
includeonly:
  name: includename
```

- `profile-common.name`는 모든 프로퍼티 파일이 공통으로 가진 프로퍼티이고, 각 파일마다 개별로 가지고있는 프로퍼티도 있다.
- test 프로파일을 활성화하고 어플리케이션을 기동하면 공통속성인 `profile-common.name`은 프로파일 프로퍼티 파일(`application-test.yml`)이 나머지를 오버라이드하여 testnameCommon이 값이 된다.
- `application-test.yml` 뿐만 아니라 디폴트 프로파일 파일인 `application.yml` 또한 읽어드리므로 `defaultonly.name`은 defaultname이 값이 된다.
- `spring.profiles.include` 프로퍼티는 포함할 다른 프로파일을 지정할 수 있다. `application-test.yml`파일에서 include 프로파일을 포함했으므로 `application-include.yml` 파일도 읽는다.
- 활성화 되지 않은 `application-dev.yml` 파일의 프로퍼티들은 읽지 않는다.

간단한 ApplicationRunner를 작성하여 프로퍼티 값을 콘솔에 찍어본다.
```java
@Component
public class Sample implements ApplicationRunner {

    @Autowired
    Environment env;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("=====================================================");
        System.out.println("spring.profiles.active : " + args.getOptionValues("spring.profiles.active"));
        System.out.println("Enviroment's Active Profile : " + Arrays.toString(env.getActiveProfiles()));
        System.out.println("defaultonly.name : " + env.getProperty("devonly.name"));
        System.out.println("testonly.name : " + env.getProperty("testonly.name"));
        System.out.println("devonly.name : " + env.getProperty("devonly.name"));
        System.out.println("profile-common.name : " + env.getProperty("profile-common.name"));
        System.out.println("includeonly.name : " + env.getProperty("includeonly.name"));
        System.out.println("=====================================================");
    }
}
```
```text
=====================================================
spring.profiles.active : [test]
Enviroment's Active Profile : [test]
defaultonly.name : null
testonly.name : testname
devonly.name : null
profile-common.name : devnameCommon
includeonly.name : includename
=====================================================
```