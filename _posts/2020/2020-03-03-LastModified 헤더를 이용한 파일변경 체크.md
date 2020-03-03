---
layout: post
title: LastModified 헤더를 이용한 파일변경 체크
tags: [LastModified, RestTemplate, ZonedDateTime, LocalDateTime]
---

# 하고싶은 작업
- 로컬에 이미지 파일을 가지고 있으며 해당 이미지 파일은 다른 곳에서 최신화가 이뤄진다.
- 원격의 이미지 파일이 최신화가 되었는지 HTTP 응답의 Lastmodified 헤더를 체크하여 로컬 이미지 파일을 최신화 한다.

# RestTemplate
- 스프링부트 웹프로젝트이므로 이미 의존성으로 들어온 RestTemplate를 이용한다. 비동기 방식이지만 어플리케이션에 크게 영향을 주지 않으므로 사용한다.
- RestTemplate으로 HTTP 요청을 날려 응답의 LAST_MODIFIED 헤더를 체크한다.

# ZonedDateTime, LocalDateTime
- HTTP 응답의 헤더는 타임존이 GMT로 되어있다.
- Asia/Seoul 지역의 시간으로 변경하여 기준일자와 비교한다.

# 테스트 코드
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class RestTemplateTest {
    @Autowired
    RestTemplateBuilder restTemplateBuilder;

    @Test
    public void RestTemplate를_이용하여_이미지_최신화() throws URISyntaxException {
        // 체크할 이미지 파일의 URL주소
        String imgSrc = "http://steamcdn-a.akamaihd.net/steam/apps/359550/capsule_sm_120.jpg";
        
                // 스프링부트의 빌더를 이용하여 RestTemplate 생성
                RestTemplate restTemplate = restTemplateBuilder.build();

        // 헤더만을 가져오도록 제공되는 메서드 사용
        HttpHeaders httpHeaders = restTemplate.headForHeaders(new URI(imgSrc));
        String lastModified = httpHeaders.get(HttpHeaders.LAST_MODIFIED).get(0);
        
                // 응답의 헤더는 GMT 시간
                System.out.println("lastModified : " + lastModified); // Fri, 21 Feb 2020 19:40:42 GMT

        // 응답의 시간에 맞는 포맷터를 지정하여 ZonedDateTime(Asia/Seoul) 객체 생성 
        ZonedDateTime lastModifiedZDT = ZonedDateTime.parse(lastModified, DateTimeFormatter.RFC_1123_DATE_TIME)
                                                     .withZoneSameInstant(ZoneId.of("Asia/Seoul"));
        System.out.println("lastModifiedZDT : " + lastModifiedZDT); // 2020-02-22T04:40:42+09:00[Asia/Seoul]

              // ZonedDateTime -> LocalDateTime
        LocalDateTime lastModifiedLDT = lastModifiedZDT.toLocalDateTime();
        System.out.println("lastModifiedLDT : " + lastModifiedLDT); // 2020-02-22T04:40:42

        // 기준일시를 생성하여 체크!
        LocalDateTime stdLDT = LocalDateTime.now().minusDays(1); // 기준일시(하루전) 생성
        if(lastModifiedLDT.isAfter(stdLDT)){ // 기준일시 이후에 변경이 일어나면
            System.out.println("다운로드(최신화) 필요함");
        }else{
            System.out.println("기존과 같음");
        }
    }
}
```