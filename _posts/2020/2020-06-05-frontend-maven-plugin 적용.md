---
layout: post
title: frontend-maven-plugin 적용
tags: [Maven, frontend-maven-plugin, 메이븐, 플러그인]
---

리액트 + 스프링부트(메이븐) 조합을 구성과정을 정리해본다.

스프링부트 프로젝트를 생성하고 별도로 `npx create-react-app projectname` 커맨드를 이용하여 리액트 프로젝트를 생성했다. 이후 스프링부트 프로젝트에서 인스톨이나 패키징시 리액트 프로젝트도 같이 작업되게 하기위해 `frontend-maven-plugin` 를 추가했다.  
일단 pom.xml 에서 의존성이 아닌 빌드와 관련된 플러그인 설정은 `<build>` 안에서 정의하는데 실제 설정한 파일을 조금씩 보면서 정리한다.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.github.eirslett</groupId>
            <artifactId>frontend-maven-plugin</artifactId>
            <version>1.10.0</version>
            <configuration>
                <!-- package.js와 frontend 설정파일이 있는 디렉터리 -->
                <workingDirectory>../yang-archive-front</workingDirectory>
                <!-- 로컬에 install될 디렉터리 -->
                <installDirectory>target</installDirectory>
            </configuration>
            <executions>
                <execution>
                    <id>install node and npm</id>
                    <goals>
                        <goal>install-node-and-npm</goal>
                    </goals>
                    <!-- optional: default phase is "generate-resources" -->
                    <phase>generate-resources</phase>
                    <configuration>
                        <nodeVersion>v14.4.0</nodeVersion>
                        <npmVersion>6.14.4</npmVersion>
                    </configuration>
                </execution>
                <execution>
                    <id>npm install</id>
                    <goals>
                        <goal>npm</goal>
                    </goals>
                    <!-- optional: default phase is "generate-resources" -->
                    <phase>generate-resources</phase>
                    <configuration>
                        <!-- optional: The default argument is actually
                        "install", so unless you need to run some other npm command,
                        you can remove this whole <configuration> section.
                        -->
                        <arguments>install</arguments>
                    </configuration>
                </execution>
                <execution>
                    <id>npm run build</id>
                    <goals>
                        <goal>npm</goal>
                    </goals>
                    <configuration>
                        <arguments>run build</arguments>
                    </configuration>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <artifactId>maven-antrun-plugin</artifactId>
            <executions>
                <execution>
                    <phase>generate-resources</phase>
                    <configuration>
                        <target>
                            <copy todir="${project.build.directory}/classes/public">
                                <fileset dir="../yang-archive-front/build"/>
                            </copy>
                        </target>
                    </configuration>
                    <goals>
                        <goal>run</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

- `<plugin>` 안에 사용하고자 하는 플러그인을 정의한다. `<groupId>`, `<artifactId>`, `<version>` 으로 사용할 플러그인이 정의된다.
- 플러그인에서 글로벌하게 설정되는 옵션값을 `<configuration>` 에 작성한다.
- `<executions>` 에서는 수행할 goal 들을 정의한다. 예를들어 `install node and npm` 의 id를 갖는 execution은 `install-node-and-npm` 골을 수행한다.(plugin 자체가 goal들의 집합이다. 여러개 골을 엮어서 수행하는 것도 가능하다.) `<goals>` 작성된 골은 `<phase>`에 작성된 페이즈에서 실행된다. `<phase>` 값을 주지 않아도 되는 경우는 디폴트로 이미 페이즈가 정의되어 있다. `<configuration>`은 마찬가지로 실행을 위한 옵션값이다.
- 위에서는 총 3개의 `<execution>`이 정의되어 있다. 모두 실행되는 기본 페이즈가 `generate-resources` 이며 install을 해보면 3개 모두 작업초기에 실행된다. `generate-resources` 페이즈가 메이븐 디폴트 라이프사이클에서 `compile` 보다 먼저 실행되는 초기 페이즈이기 때문이다.
- 뒤쪽에 추가된 `maven-antrun-plugin` 은 스프링 부트의 실행 가능한 jar를 생성시 jar파일 안에 리액트 프로젝트의 결과물도 같이 패키징되도록 추가한 플러그인이다.

### 참고
[https://github.com/eirslett/frontend-maven-plugin](https://github.com/eirslett/frontend-maven-plugin)  
[https://maven.apache.org/guides/mini/guide-configuring-plugins.html#Using_the_executions_Tag](https://maven.apache.org/guides/mini/guide-configuring-plugins.html#Using_the_executions_Tag)