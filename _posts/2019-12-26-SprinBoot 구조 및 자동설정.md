# SpringBoot 구조 및 자동설정

## 프로젝트 구조
- 스프링부트에는 main메서드를 포함하는 시작 클래스가 있고, 이 클래스에는 `@SpringBootApplication`가 마킹되어 있다.
`@SpringBootApplication`는 다수의 애노테이션으로 이루어진 메타애노테이션으로 애노테이션들이 실행되어 빈 등록 및 자동설정을 수행하며, 주요 기능을 수행하는 애노테이션은 3가지다.  
    @SpringBootApplication = @SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan

{% highlight java %}
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
...
}
{% endhighlight %}

- SpringBoot의 Bean등록은 두 단계로 나눠서 읽히게 된다.
    - 1단계 @ComponentScan : 개발자가 지정한 클래스를 빈으로 등록
    - 2단계 @EnableAutoConfiguration : 기타 라이브러리의 클래스를 빈으로 등록

- @ComponentScan : 해당 자바파일의 패키지를 기본패키지로 하위 패키지의 컴포넌트들을 모두 빈으로 등록한다. 개발자가 빈등록을 위해 애노테이션을 마킹한 클래스들이 빈으로 등록된다. 마킹에 사용되는 주요 애노테이션은 아래와 같다.
    - @Component
    - @Configuration, @Repository, @Service, @Controller, @RestController

- @EnableAutoConfiguration : @ComponentScan이 동작한 이후 자동설정이 동작한다.
    - spring.factories : @EnableAutoConfiguration 자동설정의 주요설정파일으로 모두 `spring-boot-autoconfigure` 라이브러리에 포함되어 있다.
        - 스프링부트 `spring-boot-autoconfigurer` 라이브러리의 META-INF 폴더 이하에 있는 파일이다.
        - `org.springframework.boot.autoconfigure.EnableAutoConfiguration`에 작성된 AutoConfiguration 클래스들이 모두 자동설정의 대상이 된다.
        tomcat 자동설정, DispatcherServlet 자동설정 등등... 많다.
    - spring.factories에 작성된 자동설정 대상이 되는 클래스들은 모두 @Cofiguration이 마킹되어 있다. 당연히 스프링부트 기동시 설정파일로 읽어들여 실행된다.
    - @ConditionalOnXxxYyyZzz : 모든 자동설정이 다 적용되는 것은 아니고 자동설정 클래스별로 조건에 따라 자동설정 되거나 무시된다. 
    예를들어 @ConditionalOnMissingBean 조건에는 특정 빈이 등록되어 있지 않은 경우에만 헤당 설정파일이 적용된다.

## 자동설정
스프링부트에서 의존성을 추가하고 싶을 때 주로 스프링부트에서 제공하는 starter 라이브러리를 의존성 설정파일에 추가한다.  
예를들어 Mybatis 라이브러리를 추가하고자 하면 제공되는 starter 라이브러리를 의존성설정파일(pom.xml)에 아래처럼 추가한다.  
starter 라이브러리는 단순히 의존성만 추가되는 것이 아니라 자동설정을 포함 하는데, 이것에 대해 자세히 알아본다.

{% highlight xml %}
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.1</version>
    </dependency>
{% endhighlight %}

mybatis starter 라이브러리를 기준으로 포스팅 할 예정이며 다른 라이브러리도 비슷하다.  

- `mybatis-spring-boot-starter`  
스프링부트 프로젝트의 pom.xml에 추가한 `mybatis-spring-boot-starter` 자체에는 많은 파일이 있지는 않다.  
META-INF 폴더에 pom.xml과 pom.properties 파일만 있는 정도이다.  
아래는 `mybatis-spring-boot-starter` 의 파일구성과 pom.xml 이다. starter의 pom.xml에는 사용될 다른 라이브러리와 autoconfigure(`mybatis-spring-boot-autoconfigure`) 라이브러리가 포함되어 있는 것을 볼 수 있다.

![mybatis-spring-boot-starter 구성]({{ "/assets/img/201912/SpringBoot_img1.png" | relative_url }})
{% highlight xml %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot</artifactId>
    <version>2.1.1</version>
  </parent>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <name>mybatis-spring-boot-starter</name>
  <properties>
    <module.name>org.mybatis.spring.boot.starter</module.name>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-autoconfigure</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
    </dependency>
  </dependencies>
</project>
{% endhighlight %}

- `Xxx-spring-boot-autoconfigure`

starter 라이브러리에서 참조되는 autoconfigure 라이브러리는 자동설정 클래스가 포함된다.  
(`spring.factories` 및 `XxxAutoConfiguration.class` 포함)  
스프링부트 프로젝트 기동시 메인 클래스의 `@EnableAutoConfiguration` 애노테이션이 `spring.factories` 파일을 읽어들여 자동설정을 진행한다.

![mybatis-spring-boot-autoconfigure 구성]({{ "/assets/img/201912/SpringBoot_img2.png" | relative_url }})

자동설정 중에는 스프링부트 프로젝트의 `application.properties`나 `application.yml` 같은 프로퍼티 설정파일에서 지정한 프로퍼티값을 읽어들여 활용할 수 있다. 이는 autoconfigure내의 아래 클래스들이 동작한 결과이다.

- `XxxProperties.java(MybatisProperties.java)`  
자동설정시 참조할 프로퍼티에 대해 사전에 정의해 놓은 클래스로 사용법은 아래와 같다. 

    - `XxxProperties.java(MybatisProperties.java)`에 `@ConfigurationProperties`을 마킹하면,  
    프로퍼티 설정파일에서 mybatis를 prefix로 하는 프로퍼티 값들을 읽어들여 변수에 바인딩된다.
        ```java
        @ConfigurationProperties(prefix = MybatisProperties.MYBATIS_PREFIX)
        public class MybatisProperties {
          public static final String MYBATIS_PREFIX = "mybatis";
          private String configLocation; // 프로퍼티 설정파일에서 값을 읽어들여 바인딩된다.
          // 생략
        }
        ```
    
    - AutoConfiguration 자동설정 클래스에서 @EnableConfigurationProperties(MybatisProperties.class) 애노테이션을 마킹하여 바인딩된 변수(프로퍼티 설정파일의 값)를 이용하여 자동설정 한다.
        ```java
        @org.springframework.context.annotation.Configuration
        @ConditionalOnClass({ SqlSessionFactory.class, SqlSessionFactoryBean.class })
        @ConditionalOnSingleCandidate(DataSource.class)
        @EnableConfigurationProperties(MybatisProperties.class)
        @AutoConfigureAfter({ DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class })
        public class MybatisAutoConfiguration implements InitializingBean {
        private static final Logger logger = LoggerFactory.getLogger(MybatisAutoConfiguration.class);
        private final MybatisProperties properties; // 바인딩된 properties
        // 생략
        }
        ```

요약하면, 
스프링부트에서 의존성을 추가하고 싶다면 제공되는 stater가 있는지 알아본다.  
starter는 단순히 라이브러리를 추가하는 것에 더해서  autoconfigure 라이브러리를 참조한다.  
autoconfigure는 미리 정의된 자동설정을 진행하며, 이 때 스트링부트 프로젝트의 application.properties 같은 프로퍼티 설정파일에서 값을 읽어들여 자동설정에 사용한다. 
