# Section 07. 외부 설정과 프로필2

# 외부 설정 사용 - Environment

## 외부설정

- 설정데이터
- OS 환경변수
- 자바 시스템 속성
- 커멘드 라인 인수

## 스프링이 지원하는 다양한 외부 설정 읽기

### `Environment`

```kotlin
@Configuration
class MyDataSourceEnvConfig(private val environment: Environment) {
    @Bean
    fun myDataSource(): MyDataSource {
        val url = environment["my.datasource.url"]
        val username = environment["my.datasource.username"]
        val password = environment["my.datasource.password"]
        val maxConnection = environment.getProperty("my.datasource.max-connection", Int::class.java)
        val timeout = environment.getProperty("my.datasource.timeout", Duration::class.java)
        val options = environment["my.datasource.options"]?.split(",")

        return MyDataSource(url, username, password, maxConnection, timeout, options)
    }
}
```

### `@Value` - 값 주입

값을 못 찾을 때를 대비해서 `:` 뒤에 기본 값을 작성할 수 있다.

```kotlin
@Configuration
class MyDataSourceEnvValueConfig(
    @Value("\${my.datasource.url}")
    private val url: String,
    @Value("\${my.datasource.username}")
    private val username: String,
    @Value("\${my.datasource.password}")
    private val password: String,
    @Value("\${my.datasource.max-connection:2}")
    private val maxConnection: Int,
    @Value("\${my.datasource.timeout}")
    private val timeout: Duration,
    @Value("\${my.datasource.options}")
    private val options: List<String>,
) {
    @Bean
    fun myDataSource1(): MyDataSource = MyDataSource(url, username, password, maxConnection, timeout, options)

    @Bean
    fun myDataSource2(
        @Value("\${my.datasource.url}") url: String,
        @Value("\${my.datasource.username}") username: String,
        @Value("\${my.datasource.password}") password: String,
        @Value("\${my.datasource.max-connection:2}") maxConnection: Int,
        @Value("\${my.datasource.timeout}") timeout: Duration,
        @Value("\${my.datasource.options}") options: List<String>,
    ): MyDataSource = MyDataSource(url, username, password, maxConnection, timeout, options)
}
```

* Kotlin에서 차이점! → `@Value`를 이용하면 nullable 한 타입을 쓰지 않아도 되고 자동으로 형변환까지

### `@ConfigurationProperties`
- 타입 안전한 설정 속성(`Type-safe Configuration Properties`)

외부 설정의 묶음 정보를 객체로 변환하는 기능을 제공.

쉽게 이야기해서 외부 설정을 코드로 관리할 수 있게 되는 것이다.

```kotlin
// 프로퍼티를 data class로 만들고 별도로 생성자 바인딩을 하지 않았는 데도 정상동작했다?!
// import org.springframework.boot.context.properties.bind.ConstructorBinding
// @@ConstructorBinding
@ConfigurationProperties(prefix = "my.datasource2")
data class MyDataSourcePropertiesV1(
    val url: String,
    val username: String,
    val password: String,
    val etc: Etc,
) {
    data class Etc(
        val maxConnection: Int,
        val timeout: Duration,
        val options: List<String>,
    )
}
```

```kotlin
@EnableConfigurationProperties(MyDataSourcePropertiesV1::class)
class MyDataSourceConfigV1(
    private val properties: MyDataSourcePropertiesV1,
) {

    @Bean
    fun dataSource(): MyDataSource {
        return MyDataSource(
            url = properties.url,
            username = properties.username,
            password = properties.password,
            maxConnection = properties.etc.maxConnection,
            timeout = properties.etc.timeout,
            options = properties.etc.options,
        )
    }
}
```

→ `Int` 타입인 MaxConnection을 abc로 바꾸면? (로딩 시점에 에러가 발생!)

> Description:
>
>
> Failed to bind properties under 'my.datasource2.etc.max-connection' to int:
>
> ```
> Property: my.datasource2.etc.max-connection
> Value: "abc"
> Origin: class path resource [application.properties] - 10:35
> Reason: failed to convert java.lang.String to int (caused by java.lang.NumberFormatException: For input string: "abc")
> 
> ```
>
> Action:
>
> Update your application's configuration
>

→ **위에서 `properties`을 빈 등록한 뒤에 `configuration`이를 가져와서 `dataSource`를 다시 빈 등록하는 불편한 과정이 발생했다.**

configuration class에서 `@EnableConfigurationProperties` 를 제거 한 뒤
application class에 `@ConfigurationPropertiesScan` 을 추가하면 자동으로 검색 후 주입해준다.

→ 필수값이 설정되지 않았을 경우 기본 값을 주입하기

`import org.springframework.boot.context.properties.bind.DefaultValue` 를 이용

```kotlin
@ConfigurationProperties(prefix = "my.datasource2")
data class MyDataSourcePropertiesV1(
    val url: String,
    @DefaultValue("root") val username: String,
    @DefaultValue("root") val password: String,
    val etc: Etc,
) {
    data class Etc(
        val maxConnection: Int,
        val timeout: Duration,
        val options: List<String>,
    )
}
```

→ 생성자 바인딩을 위한 annotation `@ConstructorBinding` 은 Spring 3.0 이전 버전에서 생성자 바인딩을 위해 필수로 사용되었고, 현재는 생성자가 2개 이상일 경우 사용할 생성자에 적용하면 된다.

### Type-safe Configuration Properties의  검증(validation)

maxConnection의 다른 타입이 들어오는 건 방지했지만, 값의 범위와 같은 검증은 이루어지지 않는 다.

이를 해결하기위해서 직접 개발자가 코드로 검증할 수도 있지만, 스프링에서는 미리 만들어오는 `자바 빈 검증기(java bean validation)`이라는 훌륭한 표준 검증기를 제공한다.

자바 빈 검증기를 사용하기 위해서는 `spring-boot-starter-validation` dependency를 추가해야 한다.

```kotlin
dependencies {
		implementation("org.springframework.boot:spring-boot-starter-validation")
}
```

자바 표준 검증기는 `jakarta.validation.constraints` 하위에 존재한다.

하지만 가끔 자바 표준 검증기가 아닌 하이버네이트 검증기라는 표준 검증기의 구현체에서 제공하는 것들이 있다. 대부분 큰 문제가 되진 않는다. ex) `org.hibernate.validator.constraints.time.DurationMax`

## YAML

스프링은 설정 데이터를 사용할 때 `[application.properties](http://application.properties)` 뿐만 아니라 `application.yaml(or yml)` 형식도 지원

사람이 읽기 좋은 계층 구조를 사용! 공백을 사용해 계층 구조를 만들며, 1칸을 써도 되지만 보통 2칸을 쓰고 일관되게 쓰는 게 중요하다.

구분 기호로 `:` 를 사용하여 계층구조나 값을 구분한다

> `application.properties` 와 `yml` 이 같이 있을 경우 `properties`가 우선권을 가진다.
>

## @Profile 적용하기

환경마다 설정값이 다른 것이 아니라 다른 빈을 등록해야 한다면 `@Profile`을 이용할 수 있다.

```kotlin
@Configuration
class PaymentConfig {

    @Bean
    @Profile("default")
    fun testPaymentClient(): TestPaymentClient {
        Logger.info("create testPaymentClient")
        return TestPaymentClient()
    }

    @Bean
    @Profile("prod")
    fun prodPaymentClient(): ProdPaymentClient {
        Logger.info("create prodPaymentClient")
        return ProdPaymentClient()
		}
}
```

VM 옵션: --spring.profiles.active=prod
