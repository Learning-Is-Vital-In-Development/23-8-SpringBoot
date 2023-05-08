# 외부설정과 프로필2

스프링이 지원하는 다양한 외부 설정 조회방법

- `Environment`
- `@Value`
- `@ConfigurationProperties`

# @Environment

```java

@Bean
public MyDataSource myDataSource(){

        String url=env.getProperty("my.datasource.url");
        String username=env.getProperty("my.datasource.username");
        String password=env.getProperty("my.datasource.password");
        int maxConnection=env.getProperty("my.datasource.etc.max-connection",Integer.class);
        Duration timeout=env.getProperty("my.datasource.etc.timeout",Duration.class);
        List<String> options=env.getProperty("my.datasource.etc.options",List.class);

        return new MyDataSource(url,username,password,maxConnection,timeout,options);
        }

```

- Environment를 사용하면 외부 설정의 종류와 관계없이 코드 안에서 일관성이 있게 외부 설정을 조회할 수 있다.

# @Value

- 참고로 @Value도 내부적으로 @Envitonment를 사용한다.

```java
public class MyDataSourceValueConfig {

    @Value("${my.datasource.url}")
    private String url;
    @Value("${my.datasource.username}")
    private String username;

    //필드로 주입받은 설정값 사용
    @Bean
    public MyDataSource myDataSource1() {
        return new MyDataSource(url, username);
    }

    //파라미터를 통해서 설정 값 주입
    @Bean
    public MyDataSource myDataSource2(
            @Value("${my.datasource.url}") String url,
            @Value("${my.datasource.username}") String username) {

        return new MyDataSource(url, username);
    }
}

```

# @ConfigurationProperties

스프링은 외부 설정의 묶음 정보를 객체로변환하는 기능을 제공한다. 이를 **타입 안정한 설정 속성**이라고 한다.

```java

@Data
@ConfigurationProperties("my.datasource") // 외부설정을 주입 받는 객체
public class MyDataSourcePropertiesV1 {

    private String url;
    private String username;
    private String password;
    private Etc etc = new Etc();

    @Data //getter, setter 필요
    public static class Etc {
        private int maxConnection;
        private Duration timeout;
        private List<String> options = new ArrayList<>();
    }
}
```

- 외부 설정을 주입 받을 객체를 생성하고, 필드에 외부 설정의 키 값에 맞추어 만들고,`@ConfigurationPropertie`를 사용하면 된다.

그리고 실제 사용하는 방식

```java

@Slf4j
@EnableConfigurationProperties(MyDataSourcePropertiesV1.class)
public class MyDataSourceConfigV1 {

    //주입받기
    private final MyDataSourcePropertiesV1 properties;

    public MyDataSourceConfigV1(MyDataSourcePropertiesV1 properties) {
        this.properties = properties;
    }

    @Bean
    public MyDataSource dataSource() {
        return new MyDataSource(
                properties.getUrl(),
                properties.getUsername(),
                properties.getPassword(),
                properties.getEtc().getMaxConnection(),
                properties.getEtc().getTimeout(),
                properties.getEtc().getOptions());
    }
}
```

- `@EnableConfigurationProperties(MyDataSourcePropertiesV1.class)`
    - 스프링에게 사용할 `@ConfigurationProperties` 를 지정해주어야 한다. 이렇게 하면 해당 클래스는
      스프링 빈으로 등록되고, 필요한 곳에서 주입 받아서 사용할 수 있다.
- private final MyDataSourcePropertiesV1 properties 설정 속성을 생성자를 통해 주입 받아서 사용한다.

### @ConfigurationProperties 생성자

또한 @ConfigurationProperties 생성자는 getter, setter 를 사용하는 자바빈 프로퍼티 방식이 아니라 **생성자를** 통해서 객체를 만드는 기능도지원한다.

```java

@Getter
public static class Etc {
    
    private int maxConnection;
    private Duration timeout;
    private List<String> options;

    public Etc(int maxConnection, Duration timeout, @DefaultValue("DEFAULT") List<String> options) {
        this.maxConnection = maxConnection;
        this.timeout = timeout;
        this.options = options;
    }
}
```

- 생성자를 통해서 설정 정보를 주입한다.
- `@DefaultValue`는 해당값을 찾을 수 없는 경우 기본값을 사용한다.

따라서 [`application.properties`](http://application.properties) 에 필요한 외부 설정을 추가하고, `@ConfigurationPropertis`의 생성자 주입을 통해
값을 읽으면 `setter`가 없으므로 개발자가 중간에 실수로 값을 변경하는 문제가 발생하지 않는다.

### @ConfigurationProperties 검증

`@ConfigurationPropertises`를 통해 숫자가 들어가야하는 상황에 문자가 입력되는 문제는 해결할수 있으나 숫자의 범위라던가 , 문자의 길이 같은 부분은 검증이 어렵다. 이런 문제를 해결해 주는것이
**자바 빈 검증기**이다.

자바 빈 검증기를 사용하려면 `spring-boot-starter-validation` 이 필요하다.

```java

@Getter
@ConfigurationProperties("my.datasource")
@Validated
public class MyDataSourcePropertiesV3 {

    @NotEmpty
    private String url;
    @NotEmpty
    private String username;
    @NotEmpty
    private String password;
    private Etc etc;

    public MyDataSourcePropertiesV3(String url, String username, String password, Etc etc) {
        this.url = url;
        this.username = username;
        this.password = password;
        this.etc = etc;
    }

    @Getter
    public static class Etc {
        @Min(1)
        @Max(999)
        private int maxConnection;

        @DurationMin(seconds = 1)
        @DurationMax(seconds = 60)
        private Duration timeout;

        private List<String> options;

        public Etc(int maxConnection, Duration timeout, List<String> options) {
            this.maxConnection = maxConnection;
            this.timeout = timeout;
            this.options = options;
        }
    }
}
```

- `@Validated` 어노테이션을 추가하고,  `@NotEmpty`  `@Min(1) @Max(999)` 와 같이 검증 어노테이션을 추가해서 검증을 할 수 있다.

# YAML

스프링은 설정 데이터를 사용할때 [`application.properties`](http://application.properties) 뿐만아니라 `application.yml`을 지원한다.

`YAML(YAML Ain't Markup Language)`은 사람이 읽기 좋은 데이터 구조를 목표로 한다. 확장자는 yaml , yml 이다. 주로 yml 을 사용한다.

```java

//application.properties
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```

```java

//application.yml 예시
environments:
    dev:
        url:"https://dev.example.com"
        name:"Developer Setup"
    prod:
        url:"https://another.example.com"
        name:"My Cool App"
```

# @Profile

근데 만약 설정값이 다른 정도가 아니라 각 환경마다 서로 다른 빈을 등록해야한다면 어떻게 해야할까? 
예를들어 결제 기능을 붙여야하는데 로컬 개발 환경에서는 가짜 결베 기능이 있는 스프링 빈을 등록하고, 운영환경에서는
실제 결제 기능을 제공하는 스프링 빈을 등록한다고 가정해보자 .

```java

@Slf4j
@Configuration
public class PayConfig {

    //로컬
    @Bean
    @Profile("default")
    public LocalPayClient localPayClient() {
        log.info("LocalPayClient 빈 등록");
        return new LocalPayClient();
    }

    //운영
    @Bean
    @Profile("prod")
    public ProdPayClient prodPayClient() {
        log.info("ProdPayClient 빈 등록");
        return new ProdPayClient();
    }
}
```

- `@Profile` 어노테이션을 사용하면 해당 프로필이 활성화되는 경우에만 빈을 등록한다.
    - `default` 프로필 (기본값)이 활성화 되어 있으면 `LocalPayClient` 를 빈으로 등록하고, prod 프로필이 활성화 되어있으면  `ProdPayClient`를 빈으로 등록한다.