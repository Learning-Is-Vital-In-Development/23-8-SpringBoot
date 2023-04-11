# Section 2. 웹  서버와 서블릿 컨테이너

## 웹 서버와 스프링 부트 소개

### 전통적인 방식과 부트 비교

- 전통적인 방식
    1. 톰캣 같은 WAS를 설치
    2. WAS에서 동작할 수 있도록 서블릿 스펙에 맞춰 코드 작성 후 war파일로 빌드
    3. war 파일을 이용해 설치한 WAS에 전달해서 배포
- 부트
    1. 애플리케이션 개발 후 jar파일로 빌드 / 실행하면 내부 WAS가 동작

## WAR 빌드와 배포

### **JAR와 WAR**

- JAR
    - 자바의 여러 클래스와 리소스를 묶어서 `JAR(Java Archive)`라고 하는 압축 파일을 만들 수 있다.
    - JVM 위에서 직접 실행되거나 다른 곳에서 사용하는 라이브러리로 제공
    - 직접 실행하기 위해서는 `main()` 메서드가 필요, `MANIFEST.MF`파일에 실행할 메인 메서드가 있는 클래스를 지정해주어야 함.
    - 실행 예) java -jar filename.jar
- WAR
    - 웹 어플리케이션 위에서 실행되는 파일
    - HTML과 같은 정적 리소스와 클래스 파일을 모두 포함하기 때문에 JAR와 비교해서 구조가 더 복잡
    - 구조
        - `WEB-INF` : 자바 클래스와 라이브러리 그리고 설정 정보가 들어감
            - `classes`
            - `lib`
            - `web.xml(생략가능)`
        - index.html
            - 그 외에는 HTML, CSS와 같은 정적 리소스

```bash
./gradlew build
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d522707c-d71a-413b-9294-bd476060a428/Untitled.png)

- META-INF
- WEB-INF
- index.html

### **WAR 배포하기**

- tomcat이 설치된 위치 하위에 `/webapps` 폴더에 빌드된 WAR르 파일을 이동시킨다.
- 이름을 `ROOT.war`로 변경한다.
- tomcat의 `/bin/startup.sh` 실행

  > `/logs/catalina.out` 에 로깅 정보가 쌓인다.
>


## 톰캣 설정

- IntelliJ 내부에의 Run(실행) > Edit Configuration(구성 편집) > Tomcat Local 설정 추가 및 설치 위치와 배포 정보 입력 > 아티펙트 삭제

## 서블릿 컨테이너 초기화

### **초기화 과정**

1. 서비스에 필요한 필터와 서블릿을 등록
2. 스프링을 사용한다면 스프링 컨테이너를 만들고, 서블릿과 스프링을 연결하는 디스패쳐 서블릿 등록

과거에는 `Web.xml`을 이용해서 위 설정 정보를 전달하여 초기화하였으나 현재는 자바 코드로도 가능

### **자바코드로 정보 전달하는 방법(1)**

- `ServletContrainerInitializer`라는 초기화 인터페이스의 `onStartUp()` 메서드를 구현

    ```java
    public interface ServletContainerInitializer {
        public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException;
    }
    ```

    - `Set<Class<?>> c`: 조금 더 유연한 초기화 기능 제공. `@HandlesTypes` 애너테이션을 활용
    - `ServletContext ctx`: 서블릿 컨테이너 자체의 기능 제공. 필터나 서블릿 등록에 활용
- 해당 메서드가 초기화 메서드라는 것을 전달
    - `resource > META-INF > services` 폴더에 `jakarta.servlet.ServletContainerInitializer` 파일 생성
      자신이 구현한 클래스의 전체 이름(Package + Class)을 입력

      ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0bab80f0-c002-4cba-8ccd-6efde6c29c11/Untitled.png)

- 서블릿 컨테이너는 실행되는 시점에 초기화 메서드인 `onStartUp()`을 호출해줌

### **자바코드로 정보 전달하는 방법(2)**

- 초기화를 위한 초기화 함수를 가지는 인터페이스 선언
- 해당 인터페이스를 `@HandlesTypes` 애너테이션을 이용해 새로운 `ServletContainerInitializer` 구현체에 마킹
- 구현체의 `onStartup()` 메서드 파라미터로 초기화 인터페이스의 구현체의 **클래스 정보**를 전달
- 해당 클래스 정보를 활용하여 초기화 작업 수행
    - 강의에서는 서블릿 등록을 진행하였고 등록을 위한 함수를 미리 인터페이스에 선언해두었고 인자로 `ServletContext`를 전달하여 이를 이용하여 등록 과정을 진행
    - 사용하는 이유: 애너테이션 방식은 사용하기 쉽지만 하드 코딩처럼 유연하지 못하다. 하지만 직접 프로그래밍을 할 경우 다양한 분기처리와 같은 유연하게 변경할 수 있으며 생성자에 필요한 정보를 직접 넘겨줄 수 있다.

### **애플리케이션 초기화 방식(방법2)을 사용하는 이유**

- 서블릿 컨테이너 초기화를 이용할 경우 새로운 구현체 생성 및 구현, 그리고 파일 추가라는 과정이 필요함.
  하지만 애플리케이션 초기화 방식은 애플리케이션 코드에서 작업만 하면 추가 등록이 가능!
- 또한 애플리케이션 초기화는 서블릿 컨테이너에 상관 없이 원하는 모양으로 인터페이스를 만들 수 있기 때문에 서블릿 컨테이너에 대한 의존성을 낮출 수 있음. 특히, `ServletContext`가 필요없다면 의존성을 완전 제거 가능

## 스프링 컨테이너 등록

### 등록 과정

1. 스프링 컨테이너 만들기
2. 스프링MVC 컨트롤러 스프링 컨테이너에 빈으로 등록하기
3. 디스패치 서블릿을 서블릿 컨테이너로 등록하기

## 스프링MVC 서블릿 컨테이너 초기화 지원

- 위에서 봤듯이 서블릿 컨테이너 초기화 복잡한 과정이 필요
- 스프링은 이미 이런 과정을 만들어두었고, 개발자는 어플리케이션 초기화 코드만 작성하면 됨! (아래의 인터페이스 이용)

    ```java
    public interface WebApplicationInitializer {
    	void onStartup(ServletContext servletContext) throws ServletException;
    }
    ```

  `spring-web` 라이브러리를 열어보면 아래와 같이 서블릿 컨테이너 초기화 클래스가 등록되어 있다.

  `/META-INF/services/jakarta.servlet.ServletContainerInitializer`

    ```java
    org.springframework.web.SpringServletContainerInitializer
    ```

    ```java
    @HandlesTypes(WebApplicationInitializer.class)
    public class SpringServletContainerInitializer implements ServletContainerInitializer {...}
    ```
