# WAR 배포 방식의 단점

- WAS를 별도로 설치해야한다.
- 애플리케이션 코드를 WAR로 빌드해야한다.
- 빌드한 WAR 파일을 WAS에 배포해야 한다.

자바의 main() 메서드만 실행시켜도 웹서버까지 기동할 수 있을까?

![image](https://user-images.githubusercontent.com/66561524/232171930-5db6b87c-6af2-4e4c-b940-7339dcb5738a.png)

- 왼쪽 그림은 웹 애플리케이션 서버에 WAR 파일을 배포하는 방식, WAS를 실행해서 동작한다.
- 오른쪽 그림은 애플리케이션 JAR 안에 다양한 라이브러리들과 WAS 라이브러리가 포함되는 방식


# 내장 톰캣

```java
public class EmbedTomcatServletMain {
    public static void main(String[] args) throws LifecycleException {
        System.out.println("EmbedTomcatServletMain.main");

        // 톰캣 설정
        Tomcat tomcat = new Tomcat();
        Connector connector = new Connector();
        connector.setPort(8080);
        tomcat.setConnector(connector);

        // 서블릿 등록
        Context context = tomcat.addContext("", "/");
        tomcat.addServlet("", "helloServlet", new HelloServlet());
        context.addServletMappingDecoded("/hello-servlet", "helloServlet");
        tomcat.start();
    }
}
```

- 톰캣 설정
    - 내장 톰캣 생성. 톰캣이 제공하는 커넥터 사용해서 8080 포트에 연결
- 서블릿 등록
    - 톰캣에 사용할 ‘contextPath’와 ‘docBase’를 지정해야 한다.
    - `tomcat.addServlet()` 을 통해서 등록
    - `context.addServletMappingDecoded()` 을 통해서 등록한 서블릿의 경로를 매핑
- 톰캣 시작
    - `tomcat.start()` 코드를 사용해서 톰캣을 시작

> 내장 톰캣을 개발자가 직접 다룰일은 거의 없다. 내장 톰캣이 어떤 방식으로 동작하는지 원리정도만 이해하자

# 빌드와 배포

## jar 배포

- 자바의 `main()` 메서드를 실행하기 위해서는 `jar` 형식으로 빌드해야 한다.
- jar 안에는 META-INF/MANIFETST.MF 파일에 실행할 main() 메서드의 클래스를 지정해줘야 한다.

```java
//일반 Jar 생성
task buildJar(type: Jar) {
    manifest {
        attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain'
    }
    with jar
}
```

`./gradlew clean buildJar` 를 하면 다음과 같은 에러가 발생한다.

![image](https://user-images.githubusercontent.com/66561524/232172119-f8a329e4-177c-4ea1-a5fe-5aeedafd00d2.png)

이 에러가 왜 나는지 알기 위해서는 jar에 대해서 알아야 한다. jar 파일의 압축을 풀어보자.

`jar -xvf embed-0.0.1-SNAPSHOT.jar`

`META-INF/MANIFEST.MF`

```java
Manifest-Version: 1.0
Main-Class: hello.embed.EmbedTomcatSpringMain
```
라이브러리가 전혀 없이 우리의 소스 코드만 들어가있다.

WAR를 풀었을 때는 classes 폴더와 함께 lib 폴더가 있었다. war는 내부에 라이브러리 역할을 하는 jar 파일을 포함하고 있엇다.

하지만 jar 파일은 jar 파일을 포함할 수 없다.

- WAR와 다르게 JAR 파일은 내부에 라이브러리 역할을 하는 JAR 파일을 포함할 수 없다. 포함한다고 해도 인식이 안된다. JAR 파일 스펙의 한계이다. WAR를 사용할 수도 없다. WAR는 웹 애플리케이션 서버(WAS) 위에서만 실행할 수 있다.
- 대안으로 라이브러리 jar 파일을 모두 구해서 MANIFEST 파일에 해당 경로를 적어주면 인식이 되지만 매우 번거롭고 jar 파일안에 jar 파일을 포함할 수 없기 때문에 라이브러리 역할을 하는 jar 파일도 항상 함께 가지고 다녀야 한다. 이 방법은권장하지 않는다.

## FatJar

- 대안으로 fat jar 또는 uber jar 라고 불리는 방법
- jar 안에는 jar를 포함할 수 없지만 클래스는 얼마든지 포함할 수 있다.
- 라이브러리에 사용되는 jar 풀면 class들이 나온다. 이 class들을 뽑아서 jar에 포함시키는 것이다. 그래서 수많은 라이브러리에서 나오는 class 때문에 fat jar가 나온다.

```java
//Fat Jar 생성
task buildFatJar(type: Jar) {
    manifest {
        attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain'
    }
    duplicatesStrategy = DuplicatesStrategy.WARN
    from { configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) } }
    with jar
}
```

Fat Jar 정리

장점

- Fat Jar 덕분에 하나의 jar 파일에 필요한 라이브러리들을 내장할 수 있게 되었다.
- 내장 톰캣 라이브러리를 jar 내부에 내장할 수 있게 되었다.
- 덕분에 하나의 jar 파일로 배포부터 웹 서버 설치 + 실행까지 모든 것을 단순화할 수 있다.

WAS 단점과 해결

- 톰캣 같은 WAS를 별도로 설치해야한다.
    - → WAS 를 별도로 설치하지 않고 라이브러리로 jar 내부에 포함
- 개발 환경 설정이 복잡하다
    - → main() 메서드 실행만 하면 된다.
- 배포과정이 복잡하다
    - jar 만들고 실행시키기만 하면 된다.
- 톰캣의 버전을 업데이트 하려면 톰캣을 다시 설정해야 한다.
    - → 의존성 버전 변경만 하고 빌드 후 실행하면 된다.

Fat Jar 단점

- 어떤 라이브러리가 포함되어 있는지 확인이 어렵다.
    - 모두 class로 풀려있으니 어떤 라이브러리가 사용되고 있는지 추적하기 어렵다.
- 파일명 중복을 해결할 수 없다.
    - 클래스나 리소스 명이 같은 경우 하나를 포기해야한다. 이것은 심각한 문제를 발생한다.
    - Fat Jar를 만들면 파일명이 같으므로 A, B 라이브러리가 둘 다 가지고 있는 파일 중에 하나의 파일만 선택된다. 결과적으로 나머지 하나는 포함되지 않으므로 정상 동작하지 않는다.

```java
public enum DuplicatesStrategy {
    INCLUDE,
    EXCLUDE,
    WARN,
    FAIL,
    INHERIT;

    private DuplicatesStrategy() {
    }
}
```

## 스프링 부트 실행 가능 jar

Fat Jar는 하나의 Jar 파일에 라이브러리의 클래스와 리소스를 모두 포함했다. 실행에 필요한 모든 내용을 하나의 JAR로 만들어서 배포하는 것이 가능했다.

Fat Jar의 단점

- 어떤 라이브러리가 포함되어 있는지 확인하기 어렵다.
    - 모두 class로 풀려있으니 어떤 라이브러리가 사용되고 있는지 추적하기 어렵다.
- 파일명 중복을 해결할 수 없다.
    - 클래스나 리소스명이 같은 경우 하나를 포기해야 하낟.

실행 가능 Jar

스프링 부트는 이런 문제를 해결하기 위해 jar 내부에 jar를 포함할 수 있는 특별한 구조의 jar 를 만들고 동시에 만든 jar를 내부 jar를 포함해서 실행할 수 있게 했다. 이것을 실행 가능 Jar(Executable Jar)라고 한다.

문제: 어떤 라이브러리가 포함되어 있는지 확인하기 어렵다.

- 해결: jar 내부에 jar를 포함하기 때문에 어떤 라이브러리가 포함되어있는지 쉽게 확인 가능

문제: 파일명 중복을 해결할 수 없다.

- 해결: jar 내부에 jar를 포함하기 때문에 같은 경로의 파일이 있어도 둘다 인식할 수 있다.
