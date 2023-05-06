# Section 06. 외부설정과 프로필1

## 외부 설정이란?

- 개발환경: 개발 서버, 개발 DB 사용
- 운영환경: 운영 서버, 운영 DB 사용


- 기존에는 빌드할 때 각 환경에 맞는 코드로 빌드해서 jar 파일을 만듦
    - 환경에 따라 빌드를 여러번 해야한다.
    - 개발 버전과 운영 버전의 빌드 결과물이 다르다. 따라서 개발 환경에서의 검증이 운영환경에서 다른 빌드 결과를 사용해서 예상치 못한 문제가 발생할 수 있다.
    - 각 환경에 맞춰 최종 빌드가 되어 나온 결과물이 다른 환경에서 사용할 수 없어 유연성이 떨어진다.
- 배포 환경과 무관하게 하나의 빌드 결과물을 만드는 방법
    - 설정값을 실행 시점에 각 환경에 따라 외부에서 주입
    - 빌드를 한 번만 해도 되고 개발환경에서의 빌드 결과물을 검증해서 운영에서 사용할 수 있다.

**유지보수하기 좋은 애플리케이션 개발의 가장 기본 원칙은 변하는 것과 변하지 않는 것을 분리하는 것**

- 각 환경에 따라 변하는 외부 설정값은 분리하고 변하지 않는 코드와 빌드 결과물은 유지
- 환경에 따른 유연성을 확보

## 외부설정

- 외부 설정은 일반적으로 다음 4가지 방법이 있다.
    - OS 환경 변수: OS에서 지원하는 외부 설정, 해당 OS를 사용하는 모든 프로세스에서 사용
    - 자바 시스템 속성: 자바에서 지원하는 외부 설정, 해당 JVM안에서 사용
    - 자바 커맨드 라인 인수: 커맨드 라인에서 전달하는 외부설정, 실행시 `main(args)` 메서드에서 사용
    - 외부 파일(설정 데이터): 프로그램에서 외부 파일을 직접 읽어서 사용
        - 애플리케이션에서 특정 위치의 파일을 읽도록
        - 각 서버마다 해당 파일안에 다른 설정 정보를 남겨둔다.
            - 개발서버 hello.txt: `url=dev.db.com`
            - 운영서버 hello.txt: `url=prod.db.com`

### OS 환경 변수

OS 환경 변수(OS environment variables): OS를 사용하는 모든 프로그램에서 읽을 수 있는 설정값

```java
public static void main(String[] args) {
        Map<String, String> envMap = System.getenv();
        for (String key : envMap.keySet()) {
            log.info("env {}={}", key, System.getenv(key));
        }
    }
```

OS 환경변수를 설정하고 `System.getenv()` 으로 외부 설정을 사용

- db 접근 url 같은 정보를 OS 환경변수에 설정해두고 읽으면 된다.

하지만 OS 환경변수는 이 프로그램이 아니어도 다른 프로그램에서 사용할 수 있는 전역변수의 느낌이다. 해당 애플리케이션에만 외부 설정값을 사용하고 싶으면 다른 방법을 사용해야 한다.

### 자바 시스템 속성

자바 시스템 속성(Java System properties)은 실행한 JVM 안에서 접근 가능한 외부 설정. 자바가 내부에서 미리 설정해두고 사용하는 속성들도 있다.

자바 시스템 속성은 자바 프로그램을 실행할 때 사용

- `java -Durl=dev -jar app.jar`
- `-D` VM 옵션을 통해서 `key=value` 형식을 주면 된다.

```java
public static void main(String[] args) {
        Properties properties = System.getProperties();
        for (Object key : properties.keySet()) {
            log.info("prop {}={}", key, System.getProperty(String.valueOf(key)));
        }

        String url = System.getProperty("url");
        String username = System.getProperty("username");
        String password = System.getProperty("password");

        log.info("url={}", url);
        log.info("username={}", username);
        log.info("password={}", password);

				System.setProperty("hello_key", "hello_value");
        String helloKey = System.getProperty("hello_key");
        log.info("helloKey={}", helloKey);
    }
```

`System.getProperties()`와 `System.getProperty(key)`로 속성값 조회


### 자바 커맨드 라인 인수
커맨드 라인 인수(Comman line arguments)는 애플리케이션 실행 시점에 외부 설정값을 `main(args)` 메서드의 `args` 파라미터로 전달하는 방식이다.

```java
public static void main(String[] args) {
        for (String arg : args) {
            log.info("arg {}", arg);
        }
    }
```

`java -jar app.jar url=devdb username=dev_user password=dev_pw`
- 하지만 파싱되지 않은 통 문자이다.
- 개발자가 `=`을 기준으로 데이터를 파싱해야한다. 하지만 문제가 발생할 수 있고 Map의 객체로 변환해야하는 번거로움이 있다.


**일반적인 커맨드 라인 인수**

값을 전달하는 형식이 없고 공백으로 구분한다.

- `aaa bbb` → [aaa, bbb] 값 2개
- `“hello world”` → [hello world] 공백을 연결하려면 `“`를 사용한다.

스프링은 커맨드 라인에 `--` 를 연결해서 시작하면 key=value 형식으로 정한다.

- —key=value 형식으로 사용한다.

```java
public static void main(String[] args) {
        for (String arg : args) {
            log.info("arg {}", arg);
        }

        ApplicationArguments appArgs = new DefaultApplicationArguments(args);
        log.info("SourceArgs = {}", List.of(appArgs.getSourceArgs()));
        log.info("NonOptionsArgs = {}" , appArgs.getNonOptionArgs());
        log.info("OptionsNames = {}", appArgs.getOptionNames());

        Set<String> optionNames = appArgs.getOptionNames();
        for (String optionName : optionNames) {
            log.info("option arg {}={}", optionName, appArgs.getOptionValues(optionName));
        }

        List<String> url = appArgs.getOptionValues("url");
        List<String> username = appArgs.getOptionValues("username");
        List<String> password = appArgs.getOptionValues("password");
        List<String> mode = appArgs.getOptionValues("mode");

        log.info("url={}", url);
        log.info("username={}", username);
        log.info("password={}", password);
        log.info("mode={}", mode);
    }
```

![image](https://user-images.githubusercontent.com/66561524/235309045-b9e7de42-379c-45b6-aa27-747604aa085f.png)

- 하나의 옵션에 2개 이상의 값도 들어갈 수 있다.

### 외부 파일(설정 데이터)

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class CommandLineBean {

    private final ApplicationArguments arguments;

    @PostConstruct
    public void init() {
        log.info("source {}", List.of(arguments.getSourceArgs()));
        log.info("optionNames {}", arguments.getOptionNames());
        Set<String> optionNames = arguments.getOptionNames();
        for (String optionName : optionNames) {
            log.info("option arg {}={}", optionName, arguments.getOptionValues(optionName));
        }
    }
}
```

## 외부 설정 - 스프링 통합

- 커맨드 라인 옵션 인수, 자바 시스템 속성, OS 환경변수는 외부 설정을 key=value 형식으로 사용할 수 있는 방법
- 어느 외부 설정값을 읽어야하는지에 따라서 읽는 방법이 다르다. 따라서 외부설정값을 읽는 방법이 바뀌면 코드를 모두 변경해야한다.

외부 설정값의 위치에상관없이 일관성 있이 keyvalue 형식의 외부 설정값을 읽을 수 있도록하는 방법

- 스프링은 `Environment`와 `PropertySource` 라는 추상화를 통해 해결한다.

### PropertySource

- `org.springframework.core.env.PropertySource`
- 스프링은 `PropertySource` 라는 추상 클래스를 제공하고 외부 설정을 조회하는 `XxxPropertySource` 구현체를 만들었다.
    - `CommandLinePropertySource`
    - `SystemEnvironmentPropertySource`
- 스프링은 로딩 시점에 필요한 `PropertySource` 들을 생성하고 `Environment` 에서 사용할 수 있게 연결해둔다.

### Environment

- `org.springframework.core.env.Environment`
- `Environment` 통해 특정 외부 설정에 종속되지 않고 일관성 있게 `key=value` 형식의 외부 설정에 접근할 수 있다.
    - environment.getProperty(key)를 통해 값을 조회할 수 있다.
    - Environment는 내부 과정을 통해 PropertySource 들을 접근한다.
    - 같은 값이 있는 경우 스프링은 미리 우선순위를 정해놓았다.
- 모든 외부 설정은 `Environment`를 통해 조회하면 된다.

```java
public interface Environment extends PropertyResolver {
	...
```

### 설정 데이터 (파일)

- `application.properties`, `application.yml` 도 `PropertySource`에 추가 된다.
- `Environment`를 통해 접근할 수 있다.

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class EnvironmentCheck {

    private final Environment env;

    @PostConstruct
    public void init() {
        String url = env.getProperty("url");
        String username = env.getProperty("username");
        String password = env.getProperty("password");
        log.info("env url={}", url);
        log.info("env username={}", username);
        log.info("env password={}", password);
    }
}
```

### 우선순위

- 커맨드 라인 옵션 인수 실행
    - --url=proddb --username=prod_user --password=prod_pw
- 자바 시스템 속성 실행
    - -Durl=devdb -Dusername=dev_user -Dpassword=dev_pw

우선순위 기준

- 더 유연한 것이 우선권을 가짐
    - 변경하기 어려운 파일보다 실행시 원하는 값을 줄 수 있는 자바 시스템 속성이 우선권이 높음
- 범위가 넓은 것보다 좁은 것이 우선권을 가짐
    - 자바 시스템 속성은 JVM에서 모두 접근이 가능하지만 커맨드 라인 옵션 인수는 main의 arg를 통해서 들어오기 때문에 접근 범위가 더 좁다.

### 설정 데이터1 - 외부 파일

- OS 환경변수, 자바 시스템 속성, 커맨드 라인 옵션 인수는 사용해야 하는 값이 늘어날 수록 사용하기 불편
- 대안으로 설정값을 파일에 넣어서 관리
- 애플리케이션 로딩 시점에 파일을 읽는다. 그중에서 `.properties` 파일은 `key=value` 형식을 사용해 설정값을 관리하기 적합하다
- 개발서버와 운영서버 각각에 `application.properties`라는 파일을 준비하면 애플리케이션 코드는 그대로 유지할 수 있다.
- 스프링 부트는 `application.properties` 라는 파일을 위치에 만들어만 두면 스프링이 해당 파일을 읽어서 `PropertySource` 의 구현체를 제공한다. `Environment` 를 통해서 조회 가능

### 동작 확인

- ./gradlew clean build
- build/libs
- 해당 위치에 application.properties 파일 생성
- jar 파일 실행

- 외부 설정을 별도의 파일로 관리하면 파일 자체를 관리하기 번거로운 문제가 발생
- 서버 10대면 변경사항이 있을 때 10대 서버의 설정 파일을 모두 각각 변경해야 하는 불편함이 있다.
- 설정 파일을 별도로 관리하면 설정값의 변경 이력을 확인하기 어렵다. 코드와 어떤 영향을 주고 있는지 모른다.

### 설정 데이터2 - 내부 파일 분리

- 설정 파일을 프로젝트 내부에 포함하면 변경사항 배포가 쉽다.
- 쉽게 말해 jar 하나로 설정 데이터까지 포함해서 관리한다.

- 프로젝트 안에 소스 코드와 환경에 필요한 설정 데이터도 함께 포함해서 관리한다.
    - 개발용 설정 파일: `application-dev.properties`
    - 운영용 설정 파일: `application-prod.properties`
- 빌드 시점에 개발, 운영 설정 파일을 모두 포함해서 빌드한다.
- app.jar 는 개발, 운영 두 설정 파일을 모두 가지고 배포된다.
- 실행할 때 설정 데이터를 구분해서 읽는다.
    - 개발환경이면 `application-dev.properties`
    - 운영이면 `application-prod.properties`
    - 실행할 때 외부 설정을 사용해서 dev라는 값을 제공하고 운영서버는 prod라는 값을 제공하자. 이 값을 프로필이라고 한다.
        - dev 프로필 → `application-dev.properties`
        - prod 프로필 → `application-prod.properties`

스프링은 이미 설정 데이터를 내부에 파일로 분리하고 외부 설정값(프로필)에 따라 다른 파일을 읽는 방법을 구현

### 프로필

스프링은 프로필이라는 개념을 지원한다. `spring.profiles.active` 외부 설정에 값을 넣으면 해당 프로필을 사용한다고 판단

- 프로필에 따라서 다음과 같은 규칙으로 해당 프로필에 맞는 내부 파일(설정 데이터)을 조회한다.

**커맨드 라인 옵션 인수 실행**

- -spring.profiles.active=dev

**자바 시스템 속성 실행**

- Dspring.profiles.active=dev

## 설정 데이터3 - 내부 파일 합체

- 하나의 파일로 관리하는 방법
- `application.properties`, `application.yml`은 파일 안에서 논리적으로 영역을 구분하는 방법을 제공한다.
    - `application.properties` 구분 방법 #--- 또는 !--- (dash 3)
    - `application.yml` 구분 방법 --- (dash 3)
- 프로필별로 분리가 된다.

## 우선순위 - 설정 데이터

```java
url=local.db.com
username=local_user
password=local_pw

#--
spring.config.activate.on-profile=dev
url=dev.db.com
username=dev_user
password=dev_pw

#--
spring.config.activate.on-profile=prod
url=prod.db.com
username=prod_user
password=prod_pw
```

프로필을 지정하지 않으면 default 설정으로 실행된다.

커맨드 라인 옵션 인수 실행

- -spring.profiles.active=dev

자바 시스템 속성 실행

- Dspring.profiles.active=dev

**스프링 데이터 적용 순서**

스프링은 문서를 위에서 아래로 순서대로 읽으면서 사용할 값을 설정한다.

1. 위에있는 default 데이터를 먼저 읽고 설정한다.
2. 다음 순서로 dev 관련 논리 문서를 찾는데 현재 dev 프로필이면 dev 관련 논리 문서의 값으로 대체한다. dev프로필이 아니면 무시한다.
3. prod 프로필이면 prod 논리 문서를 읽는다.
4. 논리문서에 있는 데이터만 교체가 되므로 default에 있는 값이 먼저 들어간다.

## 우선순위 - 전체

외부 설정에 대한 우선순위

[Core Features](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)

우선순위(위에서 아래)

- 설정 데이터(application.properties)
- OS 환경변수
- 자바 시스템 속성
- 커맨드 라인 옵션 인수
- `@TestPropertySource`(테스트에서 사용)

설정 데이터 우선순위

- jar 내부 `application.properties`
- jar 내부 프로필 적용 파일 `application-{profile}.properties`
- jar 외부 `[application.properties](http://application.properties)` 프로필 적용 파일
- jar 외부 프로필 적용 파일 `application-{profile}.properties`

설정 데이터 우선순위 - 스프링 공식 문서

[Core Features](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files)

우선순위 기준

- 더 유연한 것이 우선권을 가짐
    - 변경하기 어려운 파일보다 실행시 원하는 값을 줄 수 있는 자바 시스템 속성이 우선권이 높음
- 범위가 넓은 것보다 좁은 것이 우선권을 가짐
    - 자바 시스템 속성은 JVM에서 모두 접근이 가능하지만 커맨드 라인 옵션 인수는 main의 arg를 통해서 들어오기 때문에 접근 범위가 더 좁다.
