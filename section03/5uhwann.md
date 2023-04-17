# Section3: 스프링 부트와 내장 톰켓

## WAR 배포 방식의 단점과 해결

- 톰캣 같은 WAS를 별도로 설치해야 한다.
    - 해결: 톰캣과 같은 WAS가 라이브러리로 jar 내부에 포함
- 개발 환경 설정이 복잡하다
    - 웹 애플리케이션은 WAS 실행 후 WAR와 연동하기 위한 복잡한 설정이 필요
    - 해결: 단순히 `main()`메서드만 실행
- 배포 과정이 복잡하다
    - 해결: JAR를 만들고 원하는 위치에서만 실행시키면 됨
- 톰캣 버전을 변경하려면 톰캣을 다시 설치해야 한다.
    - 해결: gradle에서 내장 톰캣 라이브러리 버전만 변경후 빌드 및 실행

## 스프링 부트와 웹 서버

스프링 부트는 내장 톰켓을 사용해 빌드와 배포를 편리하게 해주고 내장 톰겟 서버를 실행하기 위한 복잡한 과정을 모두 자동으로 처리한다.

→ `org.springframework.boot:spring-boot-starter-web` 라이브러리의 의존관계를 따라가보면 내장 톰켓(`tomcat-embed-core`)이 포함된 것을 확인할 수 있다.

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

```

- 스프링 부트를 사용하면 외부 라이브러리 의존성을 사용할 때 버전정보를 명시하지 않는데, 스프링 부트가 현재 부트 버전에 가장 적절한 외부 라이브러리 버전을 자동으로 선택해 준다.

### 스프링 부트 실행 과정

1. `main()` 메서드의 `SpringApplication.run()` 호출
2. `@SpringBootApplication` 어노테이션이 있는 현재 클래스를 메인 설정정보로 지정

해당 코드 안에서 발생하는 2가지 핵심

1. 스프링 컨테이너 생성
2. WAS(내장 톰켓) 생성

**스프링 부트 내부의 스프링 컨테이너 생성 코드**

```java
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

**스프링 부트 내부의 톰캣 생성 코드**
```java
@Override
  public WebServer getWebServer(ServletContextInitializer... initializers) {
			...
      Tomcat tomcat = new Tomcat();
      ...
      Connector connector = new Connector(this.protocol);
      ...
      return getTomcatWebServer(tomcat);
}
```
