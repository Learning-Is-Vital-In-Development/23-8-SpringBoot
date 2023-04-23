# Section 3. 스프링부트와 내장톰캣
## WAR 배포 방식의 단점

기존의 배포 방식 웹 어플리케이션 서버(WAS)와 웹 어플리케이션 빌드파일(WAR)이 분리되어 있는 게 당연했다.

그래서 웹 어플리케이션을 별도로 설치하고 빌드파일도 별도로 생성한 뒤에 웹 어플리케이션에 배포

**단점**

- 별도로 설치 하는 것 자체가 단점
- 개발 환경 설정이 복잡하다.
    - 별도의 어플리케이션서버와 빌드 파일을 연동하기 위한 설정들
- 배포과정이 복잡하다

→ 톰캣을 라이브러리로 제공하는 **내장 톰캣(embed tomcat)** 기능 제공

## 내장 톰캣

- `Tomcat` 이라는 클래스를 통해서 설정 및 서블릿 등록을 바로 진행하고 띄울 수 있다.

    ```kotlin
    fun main() {
        println("EmbedTomcatServletMain.main")
    
        val tomcat = Tomcat().apply {
            // tomcat 설정
            connector = Connector().apply {
                port = 8080
            }
    
            // 서블릿 등록
            val context = addContext("", "/")
            addServlet("", "helloServlet", HelloServlet())
            context.addServletMappingDecoded("/hello-servlet", "helloServlet")
        }
    
        tomcat.start()
    }
    ```


## 빌드와 배포

- jar 파일은 jar 파일을 포함할 수 없다.
- war 파일은 내부에 라이브러리를 하는 jar를 가지고 있음.

대안으로 `FatJar`를 사용. jar를 포함할 수는 없지만 class를 다 가져와서 jar에 넣어준다.

- Fat Jar의 장점
    - 필요한 라이브러리를 jar에 내장 가능
    - WAR에서 필연적인 외부 톰캣 같은 WAS를 설치하지 않아도 됨
    - 외부 WAS와 Application 연결을 위한 불필요한 개발 환경 설정이 없어짐
    - 배포 과정이 단순화됨.
- 단점
    - 어떤 라이브러리가 포함되어 있는 지 바로 확인하기 어렵다.
    - 파일명의 중복을 해결할 수 없다.
        - 서블릿 컨테이너 초기화가 각각 라이브러리에 있다면 이걸 풀때 하나만 포함되고 하나는 포함되지 않는 문제가 발생한다.

## 스프링 부트 실행 가능 Jar

- 실행 가능 Jar(Excutable Jar): Jar 내부에 Jar를 포함할 수 있는 특별한 구조의 jar를 만들고 동시에 만든 jar를 내부 jar에 포함해서 실행할 수 있게 했다.
- jar 내부에 jar가 들어가기 때문에 Fat Jar의 문제인 라이브러리 확인 문제를 해결하고, path도 분리하여 동일한 이름의 클래스도 추가됩니다.

*실행가능한 jar는 표준이 아니고 스프링부트에서 만든 것이다.

### jar파일의 실행 정보

1. `java -jar xxx.jar` 를 실행한다.
2. `META-INF/MANIFEST.MF` 파일을 찾아 `Main-Class` 를 읽어서 `main()` 를 실행한다.

    ```
    Manifest-Version: 1.0
    Main-Class: org.springframework.boot.loader.JarLauncher
    Start-Class: hello.boot.BootApplication
    ```

    - `Main-Class`에는 내가 설정한 클래스가 아닌 스프링 부트가 빌드시에 넣어준 `JarLauncher`가 들어가 있다. `JarLauncher` 는 jar 내부의 jar를 읽어 들이는 기능이 수행한다. 스프링부트가 정의한 구조에 맞게 클래스 정보를 읽어 들인다.
    - 클래스 정보가 모두 처리가 되면 그떄 실제 사용자가 `Main-Class`로 설정했던 class인 `Start-class` 의 메인 메서드가 실행된다.

   *WAR의 구조를 본따 실행가능한 JAR를 만들었다.
