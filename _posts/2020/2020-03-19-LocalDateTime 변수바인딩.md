---
layout: post
title: Controller 메서드에서 LocalDateTime 타입의 변수 바인딩 받기
tags: [Spring, SpringBoot, Controller, LocalDateTime, JsonFormat, DateTimeFormat]
---

컨트롤러 핸들러메소드의 인자에 HTTP요청의 데이터를 자동으로 바인딩 받을 수 있다.  
바인딩 받을 변수중에 LocalDateTime 타입의 변수가 있다면, 약속된 패턴으로 스트링 데이터를 보내주거나, VO에서 별도의 애너테이션(@JsonFormat, @DateTimeFormat) 설정을 해줘야 하는데 이에 관해 정리하고자 한다.

## RequestBody(JSON 데이터를 수신하는 경우)
- `yyyy-MM-ddTHH:mm:ss` 패턴으로 JSON 데이터를 보내면 VO에 애노테이션을 지정하지 않아도 바인딩된다.

```java
// VO
public class TestVo {
    private LocalDateTime ldt;
    private String name;
}

// Controller
@RequestMapping(path = "/requestBody", method = RequestMethod.POST)
@ResponseBody
public String rbPost(@RequestBody TestVo dto){
    System.out.println("dto : " + dto);
      return "good";
}

// Test Code
@Test
public void POST_리퀘스트바디_기본패턴() throws Exception {
    mockMvc.perform(post("/requestBody")
                    .contentType(MediaType.APPLICATION_JSON_UTF8)
                    .content("{\"name\":\"yangs\", \"ldt\":\"2018-12-15T10:11:22\"}"))
            .andExpect(status().isOk())
            .andDo(print());
}
```

```text
TestVo(ldt=2018-12-15T10:11:22, name=yangs)
```

- `yyyy-MM-ddTHH:mm:ss` 패턴이 아니라면 @JsonFormat을 VO에 지정해줘야 한다.
  - @JsonFormat은 스프링부트에서 json 파싱에 사용되는 기본 라이브러리인 `Jackson`에 포함되어있다.
  - 예를들어 날짜와 시간사이에 공백이 들어간 `yyyy-MM-dd HH:mm:ss` 패턴으로 데이터를 보내는 경우  
    @JsonFormat pattern 속성에 동일하게 세팅만 되면 정상적으로 파싱 및 바인딩 된다.

```java
// VO
public class TestVo {
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Seoul")
    private LocalDateTime ldt;
    private String name;
}

// Controller
@RequestMapping(path = "/requestBody", method = RequestMethod.POST)
@ResponseBody
public String rbPost(@RequestBody TestVo dto){
    System.out.println("dto : " + dto);
      return "good";
}

// Test Code
@Test
public void POST_리퀘스트바디_기본패턴() throws Exception {
    mockMvc.perform(post("/requestBody")
                    .contentType(MediaType.APPLICATION_JSON_UTF8)
                    .content("{\"name\":\"yangs\", \"ldt\":\"2018-12-15 10:11:22\"}"))
            .andExpect(status().isOk())
            .andDo(print());
}
```

```text
TestVo(ldt=2018-12-15T10:11:22, name=yangs)
```

- 스프링 라이브러리의 @DateTimeFormat 으로 패턴을 준 경우는 제대로 동작하지 않고 오류난다.
  - Jackson에서 json파싱시 @DateTimeFormat을 바라보지 않는다.(Jackson 라이브러리 내에서 파싱하므로 스프링의 @DateTimeFormat을 알지 못한다.)

```java
// VO
public class TestVo {
  @DateTimeFormat(pattern = "yyyy/MM/dd HH:mm:ss")
  private LocalDateTime ldt;
  private String name;
}

// Controller
@RequestMapping(path = "/requestBody", method = RequestMethod.POST)
@ResponseBody
public String rbPost(@RequestBody TestVo dto){
  System.out.println("dto : " + dto);
  return "good";
}

// Test Code
@Test
public void POST_리퀘스트바디_기본패턴() throws Exception {
  mockMvc.perform(post("/requestBody")
                  .contentType(MediaType.APPLICATION_JSON_UTF8)
                  .content("{\"name\":\"yangs\", \"ldt\":\"2018-12-15 10:11:22\"}"))
          .andExpect(status().isOk())
          .andDo(print());
}
```

```text
JSON parse error: Cannot deserialize value of type `java.time.LocalDateTime` from String "2018-12-15 10:11:22"
```

**RequestBody로 Json을 바인딩 받을때는 @JsonFormat을 사용한다!**

## RequestParam
- `yyyy-MM-ddTHH:mm:ss` 패턴에 아무런 VO설정이 없으면 String을 LocalDateTime으로 변경할 수 없다는 메시지로 실패한다.(RequestBody는 아무런 설정없이 동작했다.)
- Json과 관련이 없기 때문에 @JsonFormat을 지정해도 위 케이스와 같은 오류로 실패한다.

```java
// VO
public class TestVo {
    private LocalDateTime ldt;
    private String name;
}

// Controller
@RequestMapping(path = "/requestParam", method = RequestMethod.GET)
@ResponseBody
public String rpGet(@RequestParam TestVo dto){
    System.out.println("dto : " + dto);
    return "good";
}

// Test Code
@Test
public void POST_리퀘스트파람_기본패턴() throws Exception {
    mockMvc.perform(get("/ma?name=yangs&ldt=2020-03-18T18:25:40"))
            .andExpect(status().isOk())
            .andDo(print());
}
```

```text
Failed to convert from type [java.lang.String] to type [java.time.LocalDateTime] for value '2020-03-18T18:25:40';
```

- `yyyy-MM-ddTHH:mm:ss` 패턴이 JSON변환시에는 자동으로 되어서 @DateTimeFormat에도 될것이라 예상했으나 T 문자열 때문에 오류난다.

```java
// VO
public class TestVo {
    @DateTimeFormat(pattern = "yyyy-MM-ddTHH:mm:ss")
    private LocalDateTime ldt;
    private String name;
}

// Controller
@RequestMapping(path = "/requestParam", method = RequestMethod.GET)
@ResponseBody
public String rpGet(@RequestParam TestVo dto){
    System.out.println("dto : " + dto);
    return "good";
}

// Test Code
@Test
public void POST_리퀘스트파람_기본패턴() throws Exception {
    mockMvc.perform(get("/ma?name=yangs&ldt=2020-03-18T18:25:40"))
            .andExpect(status().isOk())
            .andDo(print());
}
```

```text
nested exception is java.lang.IllegalArgumentException: Unknown pattern letter: T
```

- T대신 공백을 추가한 `yyyy-MM-dd HH:mm:ss` 패턴으로 @DateTimeFormat을 지정하면 정상 바인딩된다.  
  `yyyy/MM/dd HH-mm-ss` `yyyyMMddHHmmss` `dd-MM-yyyy HH:mm:ss` 패턴들도 @DateTimeFormat을 지정시 정상 바인딩된다.

```java
// VO
public class TestVo {
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime ldt;
    private String name;
}

// Controller
@RequestMapping(path = "/requestParam", method = RequestMethod.GET)
@ResponseBody
public String rpGet(@RequestParam TestVo dto){
    System.out.println("dto : " + dto);
    return "good";
}

// Test Code
@Test
public void POST_리퀘스트파람_기본패턴() throws Exception {
    mockMvc.perform(get("/ma?name=yangs&ldt=2020-03-18 18:25:40"))
            .andExpect(status().isOk())
            .andDo(print());
}
```

```text
TestVo(ldt=2020-03-18T18:25:40, name=yangs)
```

## ModelAttribute
- RequestParam과 같은 결과가 나온다. @DateTimeFormat 설정시에만 정상 바인딩 된다.(`yyyy-MM-ddTHH:mm:ss` 는 안된다.)

```java
// VO
public class TestVo {
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime ldt;
    private String name;
}

// Controller
@RequestMapping(path = "/modelAttribute", method = RequestMethod.POST)
@ResponseBody
public String maPost(@ModelAttribute TestVo dto){
    System.out.println("dto : " + dto);
    return "good";
}

// Test Code
@Test
public void POST_모델어트리뷰트() throws Exception {
    mockMvc.perform(post("/modelAttribute")
                        .param("name", "yangs")
                        .param("ldt", "2020-03-18 18:25:40")
                        .contentType(MediaType.APPLICATION_FORM_URLENCODED))
            .andExpect(status().isOk())
            .andDo(print());
}
```

```text
TestVo(ldt=2020-03-18T18:25:40, name=yangs)
```

## ResponseBody
- Jackson이 JSON으로 VO를 직렬화 해야하므로 @JsonFormat 설정을 따라간다.
- 만일 VO에 아무런 애너테이션도 없다면 `yyyy-MM-ddTHH:mm:ss` 패턴의 스트링 값으로 직렬화되며, @DateTimeFormat 설정이 있어도 무시하고 `yyyy-MM-ddTHH:mm:ss` 패턴으로 직렬화한다. 둘 다 설정된 경우는 당연히 @JsonFormat을 따라간다.

```java
// VO
public class TestVo {
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyyMMddHHmmss", timezone = "Asia/Seoul")
    @DateTimeFormat(pattern = "yyyy/MM/dd HH:mm:ss")
    private LocalDateTime ldt;
    private String name;

    public TestVo(String name, LocalDateTime ldt) {
        this.name = name;
        this.ldt = ldt;
    }
}

// Controller
@RequestMapping(path = "/responseBody", method = RequestMethod.GET)
@ResponseBody
public TestVo rbGet(){
    return new TestVo("yangs", LocalDateTime.now());
}

// Test Code
@Test
public void GET_리스폰스바디() throws Exception {
    mockMvc.perform(get("/responseBody")
                    .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andDo(print());
}
```

```text
MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"application/json"]
     Content type = application/json
             Body = {"ldt":"20200319005823","name":"yangs"}
```

## 정리
**@RequestBody로 JSON데이터를 LocalDateTime 변수에 바인딩하고 싶다면?**
**설정없이 `yyyy-MM-ddTHH:mm:ss` 로 JSON데이터를 보낸다. 또는 LocalDateTime 변수에 @JsonFormat을 지정한다.**
- 테스트 데이터의 패턴이 `yyyy-MM-ddTHH:mm:ss` 면서, VO 변수에 설정이 없는경우 : **성공**
- 테스트 데이터의 패턴이 `yyyy-MM-ddTHH:mm:ss` 이 아니면서, VO 변수에 @JsonFormat 설정인 경우 : **성공**
- 테스트 데이터의 패턴이 `yyyy-MM-ddTHH:mm:ss` 이 아니면서, VO 변수에 @DateTimeFormat 설정인 경우 : **실패**

**@RequestParam, @ModelAttribute, @Pathvariable로 LocalDateTime 변수에 바인딩하고 싶다면?**
**LocalDateTime 변수에 @DateTimeFormat을 지정한다.**
- 테스트 데이터의 패턴이 `yyyy-MM-ddTHH:mm:ss` 면서, VO 변수에 설정이 없는경우 : **실패**
- 테스트 데이터의 패턴이 `yyyy-MM-ddTHH:mm:ss` 이 아니면서, VO 변수에 @JsonFormat 설정인 경우 : **실패**
- 테스트 데이터의 패턴이 `yyyy-MM-ddTHH:mm:ss` 면서, VO 변수에 @DateTimeFormat 설정인 경우 : **실패**
- 테스트 데이터의 패턴이 `yyyy-MM-dd HH:mm:ss` `yyyy/MM/dd HH-mm-ss` `yyyyMMddHHmmss` `dd-MM-yyyy HH:mm:ss` 면서, VO 변수에  @DateTimeFormat 설정인 경우 : **성공**

**@ResponseBody로 VO의 LocalDateTime 변수를 JSON 데이터로 보내고 싶다면?**
**설정없이 `yyyy-MM-ddTHH:mm:ss` 으로 데이터를 보낸다. 또는 LocalDateTime 변수에 @JsonFormat을 지정한다.**
- VO에 아무런 설정이 없는경우 : `yyyy-MM-ddTHH:mm:ss` 패턴의 스트링 반환
- VO의 LocalDateTime 변수에 @DateTimeFormat 설정인 경우 : 설정이 무시되며 `yyyy-MM-ddTHH:mm:ss` 패턴의 스트링 반환
- VO의 LocalDateTime 변수에 @JsonFormat 설정인 경우 : 설정된 패턴으로 스트링 반환(`yyyy-MM-dd HH:mm:ss` `yyyy/MM/dd HH-mm-ss` `yyyyMMddHHmmss` `dd-MM-yyyy HH:mm:ss`)