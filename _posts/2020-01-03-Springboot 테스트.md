---
layout: post
title: SpringBoot 테스트
tags: [Java, Spring, SpringBoot, Test, spring-boot-starter-test, SpringBootTest, AutoConfigureMockMvc, WebMvcTest]
---
### spring-boot-starter-test
스프링부트에서는 테스트를 위해 제공하는 라이브러리가 있으며 코어 라이브러리와 Auto-Configuration을 포함하는 `Starter`를 제공한다.  
`Stater` 안에는 Spring 테스트모듈을 포함하여 JUnit, Jupiter, AssertJ, Hamcrest 등의 유용한 라이브러리가 포함되어 있다.

메이븐을 사용하다면 아래와 같이 `pom.xml`에 `spring-boot-starter-test`를 다음과 같이 추가한다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### Test Class
스프링부트 어플리케이션 테스트를 작성할 클래스에는 테스트의 성격에 따라서 제공되는 애노테이션을 조합하여 마킹할 수 있다.  
가장 기본적으로는 `@SpringBootTest`가 있는데, 이는 스프링의 `@ContextConfiguration` 애노테이션을 대체하면서 몇 가지 기능을 제공한다. (스프링에서는 테스트를 위한 환경설정을 위해 `@ContextConfiguration`를 사용한다.)

아래는 기본적인 스프링부트 테스트 클래스이다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest // 테스트를 위한 환경설정 및 부가기능을 제공한다.
public class SampleTest {

    @Test
    public void test1(){
        //
    }

    @Test
    public void test2(){
        //
    }
}
```

### @SpringBootTest의 기능
- 기본 ContextLoader로써 `SpringBootContextLoader`를 사용한다.
- 자동으로 `@SpringBootConfiguration`을 찾는다. `@SpringBootConfiguration`은 `@SpringBootApplication`(메인 클래스) 안에 있다.
- 커스텀 `Environment` 프로퍼티를 정의할 수 있다.
- 어플리케이션 구동시 설정하는 application argument를 테스트 프로그램에서 정의할 수 있다.  
  `@SpringBootTest(args="--app.test=one")`
- `WebEnvironment`를 지정할 수 있다.
    - `WebEnvironment.MOCK` : 아무런 설정이 없을시 적용되는 디폴트설정이다.  mock 서블릿 환경으로 내장톰캣이 구동되지 않는다.
    - `WebEnvironment.RANDOM_PORT` : 스프링부트를 직접 구동시킨 것처럼 내장톰캣이 구동되나 랜덤포트로 구동된다.
    - `WebEnvironment.DEFINED_PORT` : 정의된 포트로 내장톰캣이 구동된다.
    - `WebEnvironment.NONE` : `WebApplicationType.NONE`으로 구동된다.
- 테스트를 위한 `TestRestTemplate` 빈과 `WebTestClient` 빈을 등록할 수 있다.  
  만일 `WebEnvironment.MOCK` 이라면 `TestRestTemplate`과 `WebTestClient`빈은 등록되지 않는다. 실제 내장톰캣이 구동되지 않으므로 사용할 여지가 없고 어플리케이션에 요청을 보내고 싶다면 `MockMvc`로 테스트하면 되기 때문이다.

> `WebEnvironment` 설정에서 디폴트값이 MOCK 이므로 내장톰캣이 구동되지 않고 테스트가 수행되는 것이 아님에 유의한다.

### @AutoConfigureMockMvc
디폴트 설정이 서버가 실행되지 않고 Mock 환경에서 진행되므로 컨트롤러처럼 엔드포인트가 있는 테스트틑 진행할 때는 `MockMvc`를 활용한다.  
`MockMvc`가 가상의 클라이언트로 어플리케이션에 요청을 날리는 역할을 한다. `MockMvc`를 생성하는 방법에는 여러가지 있지만 테스트클래스에 `@AutoConfigureMockMvc`을 마킹하고 주입받는 것이 간단하다.

`MockMvc` 를 사용하여 mock 환경에서 컨트롤러를 테스트하는 코드이다.
```java
@Controller
public class SampleController { // 테스트 대상의 컨트롤러
    @GetMapping("/sample")
    @ResponseBody
    public String sample(){
        return "sample's return";
    }
}
```

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc // MockMvc를 생성한다.
public class SampleTest {
    @Autowired
    MockMvc mockMvc;

    @Test
    public void sampleController_Test() throws Exception {
        mockMvc.perform(get("/sample"))
                .andExpect(handler().handlerType(SampleController.class))
                .andExpect(handler().methodName("sample"))
                .andExpect(status().isOk())
                .andExpect(content().string("sample's return"))
                .andDo(print());
    }

}
```

요청을 받는 클래스나 메서드명부터 결과 상태코드, 리턴값 등 다양한 검증이 가능하다. 

### 내장톰캣 구동 테스트
Mock서블릿 환경이 아닌 내장톰캣을 구동하여 테스트를 하기 위해서는 `@SpringBootTest`에 `WebEnvironment`를 설정한다.  
`@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)`

내장 톰캣으로 구동이 되면 `MockMvc` 로는 테스트할 수 없으니 테스트를 위한 HTTP 클라이언트를 사용해야 한다. 여기서 사용할 수 있는 클라이언트가 `TestRestTemplate`다. 여기에 더해서 여러 레이어간의 동작을 보는 통합테스트가 아니라 단위 테스트인 경우에는 `MockBean`을 실제 빈 대신에 주입한 고립테스트가 가능하다.  
아래 테스트를 보자.

```java
@Controller
public class SampleController {
    @Autowired
    SampleService service; // 테스트대상이 다른 빈에 의존성을 가진다.

    @GetMapping("/sample")
    @ResponseBody
    public String sample(){
        return service.service(); // 서비스빈 호출
    }
}
```

SampleService가 어떻게 동작하는지 상관없이 SampleController를 대상으로 하는 테스트라면 `@MockBean`을 활용하여 실제빈을 대체할 수 있다.

```java
import static org.mockito.Mockito.when;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class SampleTest {
    @Autowired
    TestRestTemplate testRestTemplate;

    @MockBean
    SampleService mockSampleService;

    @Test
    public void sampleController_mockService_Test() throws Exception {
        // SampleService의 리턴을 사전에 정의한다.
                when(mockSampleService.service()).thenReturn("mock service");

        String result = testRestTemplate.getForObject("/sample", String.class);
        Assert.assertEquals(result, "mock service");
    }
}
```

Mock 서블릿 환경이 아닌 실제 내장톰캣이 랜덤포트로 구동됬으므로 `MockMvc`는 사용할 수 없다. 그대신 주입받은 테스트용 클라이언트인 `TestRestTemplate`를 활용하여 테스트 요청을 보내고 정의한 리턴값을 확인한다.

### Slice Test
`@SpringBootTest`는 자동설정을 포함한 프로젝트의 전체 빈을 모두 등록하여 테스트한다.(그러므로 무겁다.) 만일 컨트롤러처럼 MVC만 테스트하고 싶거나, JPA만 테스트하고 싶은 경우 특정 레이어의 빈만 등록하여 가볍게 테스트할 수 있도록 애노테이션을 제공하고 있다.

@WebMvcTest, @WebFluxTest, @DataJpaTest 등 제공되는 애노테이션들이 있지만 `@WebMvcTest`만 살펴본다.

### @WebMvcTest
스프링 MVC 컨트롤러에 관련된 환경구성만 진행하므로 @Controller, @ControllerAdvice, @JsonComponent, Converter, GenericConverter, Filter, HandlerInterceptor, WebMvcConfigurer, 그리고 HandlerMethodArgumentResolver 만 빈으로 등록된다.

다른 의존성이 있다면 테스트에서 빈등록 위해 명시해주거나 `MockBean`으로 대체해야 한다.

```java
@RunWith(SpringRunner.class)
@WebMvcTest(SampleController.class) // 테스트대상 클래스만 빈으로 등록한다.
public class SampleTest {
    @MockBean
    SampleService mockSampleService;

    @Autowired
    MockMvc mockMvc;

    @Test
    public void sampleController_Test() throws Exception {
        // SampleService의 리턴을 사전에 정의한다.
        when(mockSampleService.service()).thenReturn("mock service");

        mockMvc.perform(get("/sample"))
                .andExpect(handler().handlerType(SampleController.class))
                .andExpect(handler().methodName("sample"))
                .andExpect(status().isOk())
                .andExpect(content().string("mock service"))
                .andDo(print());
    }
}
```

---

요약하면
- 스프링부트에서는 테스트를 위해 Starter 라이브러리를 제공하고 있다.
- classpath에 의존성만 추가하면 자동설정까지 완료된다.
- 스프링부트 테스트에서는 `@SpringBootTest`를 활용하여 테스트를 위한 초기설정이나 환경설정을 추가/변경 할 수 있다.
- 테스트하고자 하는 대상이 다른 빈을 참조하고 있다면 Mock 빈의 사용을 고려하자.
- `@SpringBootTest`와 다르게 일부 레이어 테스트를 위한 `@WebMvcTest` 등을 제공하고 있다.