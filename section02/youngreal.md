# 2. 웹서버와 서블릿 컨테이너


### 전통적인 방식

- 서버에 톰캣같은 WAS를 설치하고 WAS에서 동작하도록 스펙에 맞춰 코드를 직접 짜고, war파일을 빌드해서 만들어야한다
- war파일을 만들면 WAS에 전달해서 배포하는방식으로 개발주기를 가졌다

최근방식

- 내장 톰캣을 포함하고있다. WAS설치, WAS와연동하는 복잡한 일을 수행하지 않아도 된다. 

### JAR

- 여러 클래스, 리소스를 묶은 압축파일
- java 명렁어로 main() 메서드가 있으면 직접 실행할수도있다.

### WAR

- WAS에 배포할때 사용하는파일
- JAR 파일이 JVM위에서 실행된다면 WAR는 WAS에서 실행된다.


### 서블릿 컨테이너 초기화 1

- resources/META-INF/services/jakarta.servlet.ServletContainerInitializer 경로에 실행할 컨테이너 패키지경로를 입력한다

- ServletContainerInitializer 인터페이스를 implements 해서 onStartUp() 메서드를 구현하는 Container클래스를 만든다

- 톰캣이 실행할때  위의 경로에 해당하는 클래스(Container)를 로딩시점에 실행해 onStartup()메서드를 실행한다.

### 서블릿 컨테이너 초기화 2(애플리케이션 초기화 방식)


서블릿을 등록하는 2가지 방법
- @WebServlet
- 프로그래밍 방식

애플리케이션 초기화, 서블릿컨테이너 초기화는 다른 개념이다.

```java
public interface AppInit {
	void onStartup(ServletContext servletContext);
}
```

```java
public class AppInitV1Servlet implements AppInit {
      @Override
      public void onStartup(ServletContext servletContext) {
      	 ServletRegistration.Dynamic helloServlet = servletContext.addServlet("helloServlet", new HelloServlet());
         helloServlet.addMapping("/hello-servlet");
    }
}
```

HelloServlet이 서블릿 컨테이너에 들어가고 매핑한 url이 호출되면 해당 url에 맞는 서블릿(강의에선 HelloServlet)이 호출된다

```java
public class HelloServlet extends HttpServlet {
	    @Override
      protected void service(HttpServletRequest req, HttpServletResponse resp) throws IOException {
      	 resp.getWriter().println("hello servlet!");
    }
}
```

  
```java
@HandlesTypes(AppInit.class)
public class MyContainerInitV2 implements ServletContainerInitializer {
      @Override
      public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException {
    	
    }
}
```
@HandleTypes를 달면 해당 인터페이스를 구현한 클래스정보를 onStartup메서드에 전달할수있다. (Set<Class<?>> c)   

AppInit인터페이스를 구현한 Servlet클래스가 c에 넘어오게된다.    

Set<Class<?>>에있던 AppInit 구현체들에게 각각 서블릿 컨테이너(ServletContext)를 건네주면서 각각 구현체들의 onStartUp메서드를 실행할수있다.


### 애플리케이션 초기화라는 개념은 왜 나왔을까

- 편리함
  - META-INF 파일에 실행할 클래스 패키지정보를 넣어주지 않아도 된다. 
  - 애플리케이션 초기화는 특정 인터페이스만 구현하면된다. 

스프링의 SpringServletContainerInitializer에는  @HandlersTypes에 WebapplicationInitializer를 넣고, 해당 인터페이스를 구현한 클래스들을 실행할수있게끔 한다(리플렉션)
