# WAR 배포 방식의 단점

### WAR 배포 방식

1. 톰캣같은 WAS를 별도 설치 
2. 애플리케이션 코드를 WAR로 빌드 
3. 빌드한 WAR 파일을 WAS에 배포

이런 배포방식은 애플리케이션을 구동하고 싶을 때 서버를 별도로 설치해야하는 구조였다. 이러한 방식에는 단점이 존재했는데


1. 톰캣같은 WAS를 별도 설치해야함.
2. 개발 환경 설정이 복잡함
3. 배포 과정이 복잡함. WAR를 만들고, 이것을 빌드해 WAS 전달해야함. 
4. 톰캣 버전을 변경하려면 톰캣을 다시 설치해야함. 

이러한 단점을 극복하기 위해 톰캣같은 WAS를 라이브러리로 내장해버렸다. 
따라서 현재는 내장 톰캣(embed tomcat)능을 제공한다.



# 내장 톰캣 - 빌드와 배포 1

자바의 main() 메서드를 실행하려면 jar의 형식으로 빌드해야함. 

```bash
//jar 빌드

./gradlew clean buildJar

//build/libs/embed-0.0.1-SNAPSHOT.jar 여기에 생성

//jar 파일 실행
java -jar embed-0.0.1-SNAPSHOT.jar

//파일 압축 풀기
jar -xvf embed-0.0.1-SNAPSHOT.jar
```

하지만 이렇게 파일을 실행하면 오류가 뜨는데,  해당 jar 파일 압축을 풀어 봤을때 스프링 라이브러리와 , 내장 톰캣 라이브러리가 전혀 보이지 않아서 오류가 발생한다. 

과거에 war의 압축을 풀었을 때, 분명 war는 내부에 라이브러리 역할을 하는 jar파일이 존재했었다. 

### jar는 jar파일을 포함할 수 없음!

war와 다르게 jar파일은 jar파일을 포함할 수 없다. 포함하더라도 인식이 안된다. 이게 Jar의 한계 따라서 다른 방법을 사용하는게 권장된다.



# 내장 톰캣 - 빌드와 배포 2

대안으로 사용되는 방법은 `FarJar`

jar파일안에는 jar파일을 포함할 수 없으나 class들은 얼마든지 포함할 수 있다. 따라서 라이브러리에 사용되는 jar를 풀어서 나온 class들을 새로 만드는 jar에 포함하면 된다 그래서 이를 `fatjar`라고 한다.

앞선방법으로 빌드하고 실행하면 실행을 했을 때 정상 동작을 확인할 수 있다. 

## Fat jar의 장단점

장점

- 하나의 jar의 파일에 필요한 라이브러리를 내장할 수 있음.
- 내장 톰캣 라이브러리를 jar 내부에 내장 가능
- 하나의 jar파일로 웹 서버 설치 + 실행까지 할 수 있음.

단점

- 모두 class로 풀려있으니 어떤 라이브러리가 사용되고 있는지 추적하기 어렵다
- 파일명 중복을 해결할 수 없다.
    - 클래스나 리소스 명이 같은 경우 하나를 포기해야 한다. 따라서 하나는 포기해야하는 상황이 온다.



# 스프링 부트  실행 가능 Jar

fat jar의 가장 큰 단점은 어떤 라이브러리가 포함되어 있는지 확인하기 어렵고, 파일명 중복을 해결할 수 없다.  이러한 `fat jar` 단점을 스프링부트는 `실행가능 Jar`로 해결했다. 

## 실행 가능 jar((Executable Jar)


`java -jar xxx.jar` 를 실행하게 되면 우선 `META-INF/MANIFEST.MF` 파일을 찾는다. 그리고 여기에 있는 Main-Class 를 읽어서 main() 메서드를 실행하게 된다.



- `Main-Class`: 우리가 기대한 main() 이 있는 hello.boot.BootApplication 이 아니라 JarLauncher 라는 전혀 다른 클래스를 실행하고 있다. JarLauncher 는 스프링 부트가 빌드시에 넣어준다. org/springframework/boot/loader/JarLauncher 에 실제로 포함되어 있다. **스프링 부트는 jar 내부에 jar를 읽어들이는 기능이 필요하다.** 또 특별한 구조에 맞게 클래스 정보도 읽어들여야 한다. 바로 JarLauncher 가 이런 일을 처리해준다. 이런 작업을 먼저 처리한 다음 Start-Class: 에 지정된 main() 을 호출한다.
- `Start-Class :` 우리가 기대한 main() 이 있는 hello.boot.BootApplication 가 적혀있다.


즉 실행과정을 정리하면, 

1. java -jar xxx.jar
2.  MANIFEST.MF 인식
3. JarLauncher.main() 실행
    - BOOT-INF/classes/ 인식 → 우리가 만든 클래스
    - BOOT-INF/lib/ 인식 → 외부 라이브러리

4. BootApplication.main() 실행