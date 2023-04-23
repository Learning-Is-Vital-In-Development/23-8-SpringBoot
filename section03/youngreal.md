# 스프링부트와 내장톰캣

## WAR 배포방식의 단점

- WAS(톰캣같은)를 별도로 설치해야한다

- 애플리케이션코드를 WAR로 빌드하고 WAS로 배포해야한다.

- 톰캣의 버전을 변경하려면 톰캣을 다시 설치해야한다.

JAR파일은 내부에 다양한 라이브러리와 WAS라이브러리가 포함되어 main() 메서드를 실행해서 동작한다. 

애플리케이션 코드를 만들기만해도 WAS가 내장되어있어서 WAS를 실행해서 코드를 배포할 필요가 없어진다.

## 내장톰캣 - 서블릿

```java
public class EmbedTomcatServletMain {

	public static void main(String[] args) throws LifecycleException {
		// 톰캣 설정
		Tomcat tomcat = new Tomcat();
		Connector connector = new Connector();
		connector.setPort(8080);
		tomcat.setConnector(connector);

		// 서블릿 등록
		Context context = tomcat.addContext("", "/");

		File docBaseFile = new File(context.getDocBase());
		if (!docBaseFile.isAbsolute()) {
			docBaseFile = new File(((org.apache.catalina.Host) context.getParent()).getAppBaseFile(), docBaseFile.getPath());
		}
		docBaseFile.mkdirs();

		tomcat.addServlet("", "helloServlet", new HelloServlet());
		context.addServletMappingDecoded("/hello-servlet", "helloServlet");
		tomcat.start();
	}
}
```

## 내장톰캣-스프링 

프레임워크를 직접 만들거 아니면 톰캣 설정해서 서블릿등록해서 톰캣을 띄울수있구나 정도만 알고있자.

```java
public class EmbedTomcatSpringMain {

	public static void main(String[] args) throws LifecycleException {
		// 톰캣 설정
		Tomcat tomcat = new Tomcat();
		Connector connector = new Connector();
		connector.setPort(8080);
		tomcat.setConnector(connector);

		// 스프링 컨테이너 생성
		AnnotationConfigWebApplicationContext applicationContext = new AnnotationConfigWebApplicationContext();
		applicationContext.register(HelloConfig.class); // 컨테이너 안에 HelloConfig를 통해 컨트롤러 생성

		// 스프링 MVC 디스패처 서블릿, 스프링컨테이너 안의 컨트롤러를 찾아서 호출할수있게된다.
		DispatcherServlet dispatcherServlet = new DispatcherServlet(applicationContext);

		// 디스패처 서블릿를 서블릿컨테이너에 등록
		Context context = tomcat.addContext("", "/");
		tomcat.addServlet("", "dispatcher", dispatcherServlet);
		context.addServletMappingDecoded("/", "dispatcher");

		tomcat.start();

		// 톰캣설정하고, 스프링컨테이너를 만들고 컨테이너안에 Controller를 생성해놓고, 중간에 DispatcherServlet을 만들어 스프링 컨테이너를 알도록 연결을 해두고 서블릿 컨테이너에 dispatcherServlet을 넣어서 "/"경로에 오게되면 dispatcher서블릿을 호출하게된다.
		// 이때 dispatcherServlet은 스프링에 있는 컨트롤러를 찾아 실행하게된다.
	}
}
```

## 내장톰캣 - 빌드와 배포 

- jar을 생성하고 실행했지만 라이브러리가 포함되지않았다. 우리가 만든 소스코드만 포함된다.
- war파일은 내부 라이브러리 역할을 하는 jar파일을 포함하고있지만 jar파일은 jar파일을 포함할수없다.

- 라이브러리들의 jar을 모두 구해서 MANIFEST파일에 경로를 적어주면 인식이되지만 매우 번거롭고, 항상 같이 패키징을 해야한다. 

## 내장톰캣 - 빌드와 배포 2

### 대안 - FatJar
- jar안에 jar은 포함할수없지만 클래스는 얼마든지 포함할수있기때문에 jar을 안에 넣는것이아닌 jar을 풀어서 나오는 클래스들을 전부 포함시키는것

#### 장점

- 필요한 라이브러리를 모두 내장할수있다.
- 하나의 jar파일로 배포, 웹서버 설치 +실행까지 단순화할수있다.
- 톰캣같은 WAS가 라이브러리로 jar내부에 포함되어있다.
- jar파일만 실행하면된다.

#### 단점
- 어떤 라이브러리가 포함되어있는지 확인하기힘들다.
- 파일명 중복을 해결할수없다.

## 스프링부트의 실행과정

스프링부트의 main메서드 안에서는 스프링컨테이너를 생성하고, WAS(내장톰캣)를 생성한다.

### 스프링 부트의 빌드와 배포

- 스프링부트는 jar내부에 jar를 포함할수있는 특별한 구조의 jar를 만들었다.

  - jar내부에 jar를 포함해 어떤 라이브러리가 포함되어있는지 확인할수있게되었다.
  
  - 경로와 파일명이 같은 파일도 둘다 인식할수있게되었다.

#### 내부구조

java -jar xxx.jar 실행 

-> META-INF/MANIFEST.MF 파일을 찾아 안에있는 Main-Class 를읽어 main()메서드를 실행하게 된다

-> Main-Class는 애플리케이션 코드의 Main이 아닌 JarLauncher가 포함되어있다 이 런쳐를 먼저 읽고, Start-Class에 지정된 애플리케이션코드의 main()을 실행한다. 
