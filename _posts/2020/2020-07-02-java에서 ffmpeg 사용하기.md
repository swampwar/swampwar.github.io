---
layout: post
title: java에서 ffmpeg 사용하기
tags: [java, ffmpeg, 썸네일]
---

스프링부트 프로젝트에서 영상 업로드 기능을 개발하던 중 영상에 대한 정보추출과 썸네일을 생성하고자 했다. 찾아보니 영상파일에 대한 
인코딩이나 정보추출, 변경 등은 ffmpeg라는 프로그램을 이용하여 한다고 하는데, 자바에서는 어떻게 사용할 수 있는지 찾아봤다.  
(일단, ffmpeg가 로컬에 설치되어 있어야 하니 brew를 통해 인스톨을 진행했다.)

먼저 프로젝트에 자바와 ffmpeg 사이에 중간다리 역할을 해주는 라이브러리(`ffmpeg-cli-wrapper`)를 내려받았다.

```xml
<dependency>
  <groupId>net.bramp.ffmpeg</groupId>
  <artifactId>ffmpeg</artifactId>
  <version>0.6.2</version>
</dependency>
```

영상파일의 메타정보 추출 및 썸네일 파일을 생성하는 `VideoFileUtils.java` 소스코드이다.  
사용하고자 하는 곳에서 빈으로 주입받아 사용한다.

```java

@Slf4j
@Component
public class VideoFileUtils {
    @Value("${yangs.ffmpeg.path}") // application.yml 파일에서 프로퍼티로 설정한다.
    private String ffmpegPath;
    @Value("${yangs.ffprobe.path}")
    private String ffprobePath;

    private FFmpeg ffmpeg;
    private FFprobe ffprobe;

    @PostConstruct
    public void init(){
        try {
            ffmpeg = new FFmpeg(ffmpegPath);
            Assert.isTrue(ffmpeg.isFFmpeg());

            ffprobe = new FFprobe(ffprobePath);
            Assert.isTrue(ffprobe.isFFprobe());

            log.debug("VideoFileUtils init complete.");
        } catch (Exception e) {
            log.error("VideoFileUtils init fail.", e);
        }
    }

    public void getMediaInfo(String filePath) throws IOException {
        FFmpegProbeResult probeResult = ffprobe.probe(filePath);

        if(log.isDebugEnabled()){
            log.debug("========== VideoFileUtils.getMediaInfo() ==========");
            log.debug("filename : {}", probeResult.getFormat().filename);
            log.debug("format_name : {}", probeResult.getFormat().format_name);
            log.debug("format_long_name : {}", probeResult.getFormat().format_long_name);
            log.debug("tags : {}", probeResult.getFormat().tags.toString());
            log.debug("duration : {} second", probeResult.getFormat().duration);
            log.debug("size : {} byte", probeResult.getFormat().size);

            log.debug("width : {} px", probeResult.getStreams().get(0).width);
            log.debug("height : {} px", probeResult.getStreams().get(0).height);
            log.debug("===================================================");
        }
    }

    public void createThumbnail(String filePath, String thumbnailPath){
        FFmpegBuilder builder = new FFmpegBuilder()
                .overrideOutputFiles(true) // 오버라이드 여부
                .setInput(filePath) // 썸네일 생성대상 파일
                .addExtraArgs("-ss", "00:00:05") // 썸네일 추출 시작점
                .addOutput(thumbnailPath) // 썸네일 파일의 Path
                .setFrames(100) // 프레임 수
                .done();
        FFmpegExecutor executor = new FFmpegExecutor(ffmpeg, ffprobe);
        executor.createJob(builder).run();
    }



}

```



참고
- [ffmpeg-cli-wrapper Examples](https://github.com/bramp/ffmpeg-cli-wrapper/wiki/Random-Examples)
- [ffmpeg-cli-wrapper API](https://bramp.github.io/ffmpeg-cli-wrapper/)
- [FFmpeg를 이용한 썸네일 이미지 추출|작성자 코기](http://blog.naver.com/PostView.nhn?blogId=ksw6169&logNo=221546693446&parentCategoryNo=&categoryNo=85&viewDate=&isShowPopularPosts=false&from=postView)
