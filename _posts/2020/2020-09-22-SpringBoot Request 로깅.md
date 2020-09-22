---
layout: post
title: SpringBoot Request 로깅
tags: [SpringBoot, Request, Logging]
---

### Spring Boot에서 Request 로깅

컨트롤러 실행 전 단계에서 Request와 메서드에 바인딩된 Argument를 로깅하고자 한다.  
각 컨트롤러의 메서드 인자로 Request를 받아 로그를 남길수도 있지만 Filter와 AOP를 활용하여 통합적으로 로깅한다.

### Filter를 등록한 이유

HTTP요청을 처리하는 아래과정에서 3번과 4번 사이에 AOP를 이용하여 로깅하고자 했다. 이 때 3번 과정이 끝나면서 AbstractJackson2HttpMessageConverter가 Request의 InputStream을 closed시켜 로깅시에 Request의 JSON값을 읽을 수가 없었다.

1. DispatcherServlet 동작
2. HandlerMapping이 요청을 처리할 Handler 선정
3. MessageConverter가 Handler(요청을 처리할 컨트롤러의 메서드) 파라미터에 값 바인딩
4. Handler 동작 

그래서 `HttpServletRequestWrapper` 를 확장한 `ReadableHttpServletRequestWrapper`를 생성하여 스트림이 닫혀도 계속 getInputStream()호출시 새로운 스트림을 생성하여 반환하도록 했다.  이 클래스는 Filter에서 기존 Request 객체를 이용해 생성하고 이를 대체한다.

```java
public class ReadableHttpServletRequestWrapper extends HttpServletRequestWrapper {
    private byte[] body;

    public ReadableHttpServletRequestWrapper(HttpServletRequest request) throws IOException {
        super(request);

        ServletInputStream inputStream = request.getInputStream();
        body = inputStream.readAllBytes();
        inputStream.close();
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        ByteArrayInputStream is = new ByteArrayInputStream(body);
        return new ServletInputStream() {
            @Override
            public boolean isFinished() {
                return false;
            }

            @Override
            public boolean isReady() {
                return false;
            }

            @Override
            public void setReadListener(ReadListener listener) {

            }

            @Override
            public int read() throws IOException {
                return is.read();
            }
        };
    }

    public String readBody(){
        StringBuilder sb = new StringBuilder("\n");
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(new ByteArrayInputStream(body)));
        bufferedReader.lines().forEach(str -> sb.append(str).append("\n"));
        return sb.toString();
    }
}
```

```java
@Slf4j
@WebFilter(urlPatterns = "/*")
public class MyWebFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest mRequest = (HttpServletRequest) request;
        HttpServletRequest yRequest = new ReadableHttpServletRequestWrapper(mRequest);
        chain.doFilter(yRequest, response); // 기존 Request 객체를 대체한다.
    }

    @Override
    public void destroy() {
    }
}
```

### AOP를 이용하여 컨트롤러 시작 전 로깅

```java
@Slf4j
@Component
@Aspect
public class LoggerAspect {
    @Pointcut("execution(* wind.yang.yangsArchive..*Controller.*(..))")
    public void loggerPointCut(){

    }

    @Around("loggerPointCut()")
    public Object methodLogger(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        Object result = proceedingJoinPoint.proceed();

        if(log.isDebugEnabled()){
            ReadableHttpServletRequestWrapper request = (ReadableHttpServletRequestWrapper) ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest(); // request 정보를 가져온다.
            String controllerName = proceedingJoinPoint.getSignature().getDeclaringType().getSimpleName();
            String methodName = proceedingJoinPoint.getSignature().getName();

            log.debug("=== Request Logging Before Process Controller =======================");
            log.debug("RequestUri : [{}], HttpMethod : [{}]", request.getRequestURI(), request.getMethod());
            log.debug("controllerName : [{}], methodName : [{}]", controllerName, methodName);
            log.debug("body : {}", request.readBody());

            Object[] args = proceedingJoinPoint.getArgs();
            log.debug("=== Args Logging Before Process Controller ==========================");
            for(Object obj : args){
                log.debug(String.valueOf(obj));
            }
            log.debug("=====================================================================");
        }

        return result;
    }
}
```

### 참고

- [Spring에서 Request를 우아하게 로깅하기](https://taetaetae.github.io/2019/06/30/controller-common-logging/)
- [13. 스프링부트 MVC - Filter 설정](https://linked2ev.github.io/gitlog/2019/09/15/springboot-mvc-13-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-MVC-Filter-%EC%84%A4%EC%A0%95/)