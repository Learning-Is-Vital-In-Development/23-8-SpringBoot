# Section 06. 외부 설정과 프로필1

## 외부설정이란?

하나의 애플리케이션을 여러 다른 환경에서 사용.

- 개발 환경, 운영 환경, 테스트 환경 등

이러한 다양한 환경에서는 서로 다른 설정값이 필요할 수 있음.

### 기존 빌드 방식의 단점과 해결책

- 코드 내부에 필요한 값을 넣어 각각 빌드하여 배포한다.

  단점

    - 환경에 따라 빌드를 여러번 해야 함
    - 빌드 결과물이 다르기 때문에 예상치 못한 문제가 발생할 수 있음
    - 향후 다른 환경이 필요해지면 또 빌드를 해야하기 때문에 유연성이 떨어진다.
- 빌드를 한번만 하고 각 환경에 맞춰 **실행 시점에 외부 설정값을 주입**

  기존 방식 단저

    - ~~환경에 따라 빌드를 여러번 해야 함~~ → 빌드 1번만 해도 됨
    - ~~빌드 결과물이 다르기 때문에 예상치 못한 문제가 발생할 수 있음~~
      → 한번만 빌드하여 결과물이 같음.
    - ~~향후 다른 환경이 필요해지면 또 빌드를 해야하기 때문에 유연성이 떨어진다.~~
      → 외부 설정값만 변경하여 다양한 환경에서 사용할 수 있음

**유지보수하기 좋은 애플리케이션 개발의 가장 기본 원칙은 변하는 것과 변하지 않는 것을 분리하는 것**

## 외부 설정을 주입하는 방법

여러 가지 설정 값들을 통해 주입이 가능하다.

### OS 환경 변수

: OS 에서 지원하는 외부 설정, 모든 프로그램에서 읽을 수 있어 범위가 가장 넓음

```kotlin
fun main() {
    val envMap = System.getenv()

    for (key in envMap.keys) {
        Logger.info("env $key = ${envMap[key]}")
    }
}
// 실행결과
// INFO: env PATH = /opt/homebrew/bin:/opt/homebrew/sbin:/Users/dojinyou/.nvm/versions/node/v14.21.2/bin:/Users/dojinyou/.docker/bin:/usr/local/bin:/System/Cryptexes/App/usr/bin:/usr/bin:/bin:/usr/sbin:/sbin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/local/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/appleinternal/bin:/Users/dojinyou/Library/Application Support/JetBrains/Toolbox/scripts
// 2023-04-06 00:18:56 [main] com.dojinyou.inflearn.w4.external.OsEnvKt.main()
// INFO: env MANPATH = /opt/homebrew/share/man::
// 2023-04-06 00:18:56 [main] com.dojinyou.inflearn.w4.external.OsEnvKt.main()
// INFO: env SSH_SOCKET_DIR = ~/.ssh
// ...
```

- `System.getevn()` 를 사용하여 전체 OS 환경 변수를 `Map` 으로 조회

여러 프로그램에서 사용하는 것이 맞을 때도 있지만, 어플리케이션을 사용하는 자바 어플리케이션에서만 사용할 경우에는 적절하지 않다.

### 자바 시스템 속성

: 자바에서 지원하는 외부 설정, 해당 JVM 안에서 사용

### 자바 커맨드 라인 인수

: 커맨드 라인에서 전달하는 외부 설정, 실행 시 `main(args)` 메서드 사용

### 외부 파일 - 설정 데이터

: 프로그램 외부 파일을 직접 읽어서 사용

- 특정 위치에 파일을 읽도록 처리
- 각 서버마다 다른 파일안에 다른 설정 정보를 남겨둔다.

### 외부 설정 - 자바 시스템 속성

: 자바 시스템 속성(java System Properties)는 실행한 JVM 안에서 접근 가능한 외부 설정. 추가로 자바가 미리 설정해두고 사용하는 것들도 있다.

- 자바를 실행할 때 아래와 같이 사용한다.
    - `java - Durl=dev -jar app.jar`
    - `-D` VM 옵션을 통해서 `key=value` 형식을 주면 된다.
    - `-D` 옵션이 `-jar` 옵션보다 앞서야 한다.

  → 코틀린은 `System.getProperty()` 를 호출하면 `key`에 `key=value`가 함께 들어온다.


### 외부 설정 - 커멘트 라인 인수

: 커멘드 라인 인수(Command Line  arguments)는 실행 시점에 외부 설정 값을 `main(args)` 메서드의 `args` 파라미터로 전달하는 방법이다.

- `java -jar app.jar dataA dataB`
- 필요한 데이터는 마지막 위치에 스페이스로 구분해서 전달
- `key-value` 형식이 아니라 단순한 `String` 값으로 전달된다.

  → 따라서 결국 파싱 후 `Map`과 같은 형식으로 변환하여 직접 개발해야한다.

- IntelliJ 에서는 빌드 구성 정보에서 프로그램 인수를 이용해 사용

  ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3ba50dd1-1476-46b3-be7d-73a6442dd2e1/Untitled.png)


### 외부 설정 - 커멘드 라인 옵션 인수

: 커맨드 라인 인수를 `key=value` 형식으로 구분하는 걸 선호하는 데 이걸 편하게 사용하도록 만든 spring만의 표준 방식

- 스프링은 커맨드 라인에 dash 2개를 연속으로 연결해서 `—-` 로 시작하면 `key=value` 형식으로 정하고 이것을 커맨드 라인 옵션 인수라 한다.
- `—-key=value` 형식으로 사용
    - `—-username=userA —-username=userB` 로 같은 key 에 여러 값을 지정할 수도 있음.
- 이것을 이용하기 위해서 `ApplicationArguments` 라는 interface를 구현해야함.
    - `DefaultApplicationARguments`라는 구현체가 이미 있고 `args`를 인자로 생성자에 넘겨준다.

        ```kotlin
        fun main(args: Array<String>) {
        	val appArgs: ApplicationArguments = DefaultApplicationArguments(*args)
        	println("SourceArgs = ${appArgs.sourceArgs.toList()}")
        	println("NotOptionArgs = ${appArgs.nonOptionArgs}")
        	println("OptionArgs = ${appArgs.optionNames}")
        	
        	val optionNames = appArgs.optionNames
        	for (optionName in optionNames) {
        	    println("option arg $optionName = ${appArgs.getOptionValues(optionName)}")
        	}
        }
        ```

- 옵션 인수는 하나의 키에 여러 값을 포함할 수 있기 때문에 리스트를 반환한다.
- 이 기능은 자바의 표준 기능이 아닌 스프링이 편리함을 위해 제공하는 기능이다.

### 커멘드 라인 옵션 인수와 스프링부트

- 스프링부트는 `ApplicationArgument`의 `DefaultApplicationArguments`를 빈으로 등록하여 준다.
- 따라서 필요한 곳에서 주입 받아서 사용할 수 있다.

### 스프링 통합

- `키=밸류` 형태로 입력되지만 입력되는 장소에 따라 값을 가져오는  코드가 달라짐
- 이를 해결하기 위해 `Enviroment`로 추상화하여 각각의 소스에서 들어오는 것들을 가지고 있음

    ```kotlin
    @Component
    class EnviromentCheck(private val env: Environment) {
    
        @PostConstruct
        fun init() {
            val url = env["url"]
            val username = env["username"]
            val password = env["password"]
            println("url = $url")
            println("username = $username")
            println("password = $password")
        }
    }
    ```

- 커맨드라인 인수와 자바 시스템 속성 모두 `Enviroment` 를 통해서 동일한 방법으로 읽을 수 있음.

우선순위

- 커맨드라인 옵션 인수와 자바 시스템 속성 중 커맨드 라인 옵션 인수가 우선순위가 높다.
- 기본적인 원칙은 단순하다.
    1. 더 유연한 것이 우선권을 가진다.
    2. 범위가 좁고 디테일 할수록 더 우선순위를 가진다.

### 설정 데이터 - 외부 파일

`application.properties` 파일을 각각의 환경에 다른 값을 작성하고 각 환경에서 해당 파일을 단순히 읽어서 사용하게 한다.

> 해당 파일은 `src/main/resource/application.properties`가 아닌 jar 빌드 외부의 파일이다.
>

스프링은 해당 설정 데이터를 읽기 위해서 `PropertySource` 의 구현체를 제공하고 `Environment` 으로 읽어서 사용할 수 있다.

해당 파일을 프로젝트 외부에 있기 때문에 별도의 관리가 필요하다는 단점이 있다.

### 설정 데이터 - 내부 파일 분리

별도 관리의 문제를 해결하는 방법으로 내부의 소스코드와 함께 관리하는 것이다.

관리방법

1. 소스코드에 개발환경에 맞는 설정 데이터 파일을 구성한다.
    - 설정파일 네이밍: `application-{profile_name}.properties`
    - 개발환경 설정 파일: `application-dev.properties`
    - 운영환경 설정 파일: `application-prod.properties`
2. 설정 파일을 함께 빌드 하고 배포한다.
3. 실행 시점에서 활성화된 프로필(`spring.profiles.active`)를 읽어서 환경을 구분하고 그에 맞는 설정파일을 읽어 사용한다.

위파일이 분리되어 있어서 한 눈에 전체가 들어오지 않는 다는 단점이 있다?

### 설정 데이터 - 내부 파일 합체

파일을 분리하면서 한눈에 들어오지 않는 부분을 스프링은 물리적 한 파일 속에서 논리적으로 구분해서 사용할 수 있도록 지원하고 있다.

논리적으로 구분하는 방법

- 하나의 파일(`application.properties`) 속에서 구분자를 이용해서 설정 데이터를 분리한다.
    - 구분자는 `#---` 또는 `!---` 를 이용한다.(yml은 `---`)
    - 구분자 전후로 주석이 있다면 정상적으로 동작하지 않을 수 있으니 주의합니다.
- 구분된 설정파일에 활성화시킬 프로필값을 추가한다.
    - ex) `spring.config.activate.on-profile=dev`
    - 별도의 활성 프로필 설정이 없다면 항상 입력된다

### 우선순위 - 설정데이터

활성화될 프로필값을 주지 않으면 `default`라는 기본 프로필 값이 활성화된다.

스프링은 항상 위에서 아래로 읽으면서 설정한다.

### 우선순위 - 전체

자주 사용하는 우선순위(위에서 아래로 적용되고 아래로 갈수록 우선순위가 높다)

1. 설정데이터(`application.properties`)
    1. jar 내부 `application.properties`
    2. jar 내부 프로필 적용 파일 `application-{profile}.properties`
    3. jar 외부 `application.properties`
    4. jar 외부 프로필 적용 파일 `application-{profile}.properties`
2. OS 환경변수
3. 자바 시스템 속성(`-D{key}={value}`)
4. 커맨드 라인 옵션 인수(`--key=value`)
5. `@TestProeprtySource`(테스트에서 사용)
