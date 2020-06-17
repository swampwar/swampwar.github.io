---
layout: post
title: View와 ViewResolver
tags: [스프링, Spring, View, ViewResolver]
---

스프링의 컨트롤러에서 핸들러메서드는 ModelAndView 객체에 View이름을 문자열로 초기화하여 리턴하거나 String 으로 View이름을 리턴한다. 메서드에서 "hello" 라는 문자열만 리턴해도 맵핑된 "hello.jsp", "hello.html" 등이 브라우저에 랜더링 되는것은 중간에 View 와 ViewResolver 내부적으로 동작하기 때문이다.  
HTTP 요청에 대한 업무로직 처리는 핸들러메서드가 집중하여 담당하고 뷰와 관련된 로직은 따로 책임을 가진 프로그램에서 처리하기 위함이다.

```java
@Controller
public class TestController {
    @GetMapping("/test")
    public String test(){
        return "/test"; // 문자열을 리턴해도 test.html, test.jsp 등이 맵핑되어 그려진다.
    }
}
```

스프링은 Front Controller 방식으로 모든 요청을 `DispatcherServlet`이 먼저 받아 처리한다. `DispatcherServlet`에서 HTTP 요청을 해석하여 맵핑된 핸들러메서드를 호출해 로직을 처리하고 핸들러메서드에서 리턴된 뷰정보(위 소스에서는 "/test"라는 문자열)를 다시 해석하여 브라우저에 그려질 화면을 응답으로 내보낸다.

핸들러메서드에서 리턴된 뷰정보(뷰의 이름을 문자열로 리턴)를 해석하는 것이 `ViewResolver`, 리졸버에의해 해석된 뷰 자체를 `View`로 이해하자.

`ViewResolver`는 인터페이스로 다양한 구현체가 있고, 핵심 메서드는 문자열을 인자로 받아 해석된 `View`를 리턴하는 `resolveViewName()`이다.

```java
public interface ViewResolver {
	  @Nullable
	  View resolveViewName(String viewName, Locale locale) throws Exception;
}
```

View도 인터페이스로 다양한 구현체가 있고, 핵심 메서드는 처리된 데이터(model)를 가지고 브라우저에 그려질 최종 결과물(html)을 전달하는 render() 이다. 뷰를 jsp로 구성하면 JstlView, 타임리프를 사용한다면 ThymeleafView 등 환경구성에 따라 다양한 View 구현체들이 사용된다.

```java
public interface View {
	void render(@Nullable Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

### InternalResourceViewResolver

ViewResolver의 구현체중 가장 많이 사용되는 것이 InternalResourceViewResolver이다. DispatcherServlet이 디폴트로 가지고 있는 뷰리졸버로 prefix, suffix 속성을 이용하여 실제 View를 해석한다.

예를들어 핸들러메서드에서 요청 처리후 "home" 이라는 문자열을 리턴했다고 하면,

1. DispatcherServlet은 리턴된 문자열을 InternalResourceViewResolver에게 전달하여 해당하는 View가 있는지 물어본다.
2. 설정된 prefix가 "/WEB-INF/views/" 이고 suffix가 ".jsp" 라면 인자로 전달된 문자열과 조합하여 "/WEB-INF/views/home.jsp"를 찾는다.
3. 해당 path의 jsp파일이 있으면 브라우저 랜더링을 위해 적절한 View 객체(InternalResourceView 객체 등)를 리턴한다.
4. DispatcherServlet은 리턴된 View의 render()함수를 이용하여 브라우저에게 화면(html)을 전달한다.

DispatcherServlet은 디폴트로 여러개의 ViewResolver를 가지고있다. 이들 사이에는 설정된 우선순위가 있으며 높은 우선순위부터 돌아가면서 View를 리턴하는지 검사하고 모두 null을 리턴하면 View를 찾을 수 없다고 판단 후 404에러가 떨어진다.

참고용으로 XML방식으로 구성된 InternalResourceViewResolver의 예시이다. 리턴할 View의 클래스, prefix, suffix, order 등이 설정된 것을 볼 수 있다. 스프링부트로 프로젝트를 구성했다면 프로퍼티설정파일(application.yaml 등)에 설정이 가능하다.

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
  <property name="prefix" value="/WEB-INF/jsps/" />
  <property name="suffix" value=".jsp" />
  <property name="order" value="2" />
  <property name="viewNames" value="*jsp" />
</bean>
```

### 참고

[What is the Role of InternalResourceViewResolver in Spring MVC? Interview Question](https://javarevisited.blogspot.com/2017/08/what-does-internalresourceviewresolver-do-in-spring-mvc.html)