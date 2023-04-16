# Section3: 스프링 부트와 내장 톰켓

## 0.WAR 배포 방식의 단점

- WAS를 별도로 설치해야 한다.

- 개발 환경 설정이 복잡

- 배포 과정이 복잡

- 톰캣 버전을 변경하려면 톰캣을 다시 설치해야 한다.
    - Sol: gradle에서 내장 톰캣 기능 이용.
    
## 1.내장 톰캣 설명 (1)

1. embed-start 의 폴더 이름을 embed 로 변경하자.     
2. 프로젝트 임포트     
3. File Open 해당 프로젝트의 build.gradle 을 선택하자.      
4. 그 다음에 선택창이 뜨는데, Open as Project를 선택하자.     
build.gradle 확인    
```
plugins {     
 id 'java'     
}     
group = 'hello'     
version = '0.0.1-SNAPSHOT'     
sourceCompatibility = '17'     
repositories {    
 mavenCentral()    
}    
dependencies {    
 //스프링 MVC 추가    
 implementation 'org.springframework:spring-webmvc:6.0.4'    
 //내장 톰캣 추가    
 implementation 'org.apache.tomcat.embed:tomcat-embed-core:10.1.5'     
}    
tasks.named('test') {     
 useJUnitPlatform()       
}      
//일반 Jar 생성    
task buildJar(type: Jar) {     
 manifest {     
 attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain'     
 }     
 with jar     
}     
//Fat Jar 생성      
task buildFatJar(type: Jar) {     
 manifest {     
 attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain'     
 }    
 duplicatesStrategy = DuplicatesStrategy.WARN    
 from { configurations.runtimeClasspath.collect { it.isDirectory() ? it :    
zipTree(it) } }     
 with jar     
}     
```
- tomcat-embed-core : 톰캣 라이브러리이다. 톰캣을 라이브러리로 포함해서 톰캣 서버를 자바 코드로
실행할 수 있다.    

## 1.내장 톰캣 설명 (2) 서블릿
이제 본격적으로 내장 톰캣을 사용해보자.    내장 톰캣은 쉽게 이야기해서 톰캣을 라이브러리로 포함하고
자바 코드로 직접 실행하는 것이다.     
```
public class EmbedTomcatServletMain {    
 public static void main(String[] args) throws LifecycleException {    
 System.out.println("EmbedTomcatServletMain.main");    
 //톰캣 설정    
 Tomcat tomcat = new Tomcat();    
 Connector connector = new Connector();    
 connector.setPort(8080);    
 tomcat.setConnector(connector);    
 //서블릿 등록    
 Context context = tomcat.addContext("", "/");    
 tomcat.addServlet("", "helloServlet", new HelloServlet());    
 context.addServletMappingDecoded("/hello-servlet", "helloServlet");    
 tomcat.start();   
 }   
}   
```
- 톰캣 설정     
내장 톰캣을 생성하고, 톰캣이 제공하는 커넥터를 사용해서 8080 포트에 연결한다.    
- 서블릿 등록     
톰캣에 사용할 contextPath 와 docBase 를 지정해야 한다. 이 부분은 크게 중요하지 않으므로 위     
코드와 같이 적용하자.    
tomcat.addServlet() 을 통해서 서블릿을 등록한다.    
context.addServletMappingDecoded() 을 통해서 등록한 서블릿의 경로를 매핑한다.    
- 톰캣 시작    
tomcat.start() 코드를 사용해서 톰캣을 시작한다.    
등록한 HelloServlet 서블릿이 정상 동작하는지 확인해보자.    
- 실행    
EmbedTomcatServletMain.main() 메서드를 실행하자.    
http://localhost:8080/hello-servlet     
- 결과    
hello servlet!    
내장 톰캣을 사용한 덕분에 IDE에 별도의 복잡한 톰캣 설정 없이 main() 메서드만 실행하면 톰캣까지    
매우 편리하게 실행되었다. 물론 톰캣 서버를 설치하지 않아도 된다    
## 2. 스프링 부트와 웹 서버   

스프링 부트는 내장 톰켓을 사용해 빌드와 배포를 편리하게 해주고 내장 톰겟 서버를 실행하기 위한 복잡한 과정을 모두 자동으로 처리한다.

→ `org.springframework.boot:spring-boot-starter-web` 라이브러리의 의존관계를 따라가보면 내장 톰켓(`tomcat-embed-core`)이 포함된 것을 확인할 수 있다.

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
```

- 스프링 부트를 사용하면 외부 라이브러리 의존성을 사용할 때 버전정보를 명시하지 않는데, 스프링 부트가 현재 부트 버전에 가장 적절한 외부 라이브러리 버전을 자동으로 선택해 준다.

### 2. 스프링 부트 실행 과정

1. `main()` 메서드의 `SpringApplication.run()` 호출
2. `@SpringBootApplication` 어노테이션이 있는 현재 클래스를 메인 설정정보로 지정

해당 코드 안에서 발생하는 2가지 핵심

1. 스프링 컨테이너 생성
2. WAS(내장 톰켓) 생성

# 스프링 컨테이너 생성

```
class ServletWebServerApplicationContextFactory implements
  ApplicationContextFactory {
    ...
    private ConfigurableApplicationContext createContext() {
      if (!AotDetector.useGeneratedArtifacts()) {
        return new AnnotationConfigServletWebServerApplicationContext();
      }
    return new ServletWebServerApplicationContext();
}
```
- `new AnnotationConfigServletWebServerApplicationContext()` : 스프링 컨테이너 생성
# 톰캣 생성 
```
@Override
  public WebServer getWebServer(ServletContextInitializer... initializers) {
			...
      Tomcat tomcat = new Tomcat();
      ...
      Connector connector = new Connector(this.protocol);
      ...
      return getTomcatWebServer(tomcat);
}
