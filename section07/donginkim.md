## 외부 설정 사용 - Environment

외부 설정들은 스프링이 제공하는 Environment 통해 일관된 방식으로 조회할 수 있다.

- 설정 데이터(application.properties)
- OS 환경변수
- 자바 시스템 속성
- 커맨드 라인 옵션 인수

스프링이 지원하는 다양한 외부 설정 조회 방법

- Environment
- `@Vaule` 값 주입
- `@ConfigurationProperties` - 타입 안전한 설정 속성

스프링은 다양한 타입들에 대해 기본 변환 기능을 제공한다.

[Core Features](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.conversion)

정리
- `Environment`는 향후 외부 설정 방식이 달라지더라도 애플리케이션 코드를 그대로 유지할 수 있다.

단점

- `Environment`를 직접 주입받고 `env.getPrperty(key)`를 통해서 값을 꺼내는 과정을 반복해야 한다는 단점이 있다.
- 스프링은 `@Value`를 통해 설정값을 주입받는 편리한 기능을 제공한다.

## 외부설정 사용 - @Value

```java
@Slf4j
@Configuration
public class MyDataSourceValueConfig {

    @Value("${my.datasource.url}")
    private String url;
    @Value("${my.datasource.username}")
    private String username;
    @Value("${my.datasource.password}")
    private String password;
    @Value("${my.datasource.etc.max-connection}")
    private int maxConnection;
    @Value("${my.datasource.etc.timeout}")
    private Duration timeout;
    @Value("${my.datasource.etc.options}")
    private List<String> options;

    @Bean
    public MyDataSource myDataSource1() {
        return new MyDataSource(url, username, password, maxConnection, timeout, options);
    }

    @Bean
    public MyDataSource myDataSource2(
    @Value("${my.datasource.url}")
    String url,
    @Value("${my.datasource.username}")
    String username,
    @Value("${my.datasource.password}")
    String password,
    @Value("${my.datasource.etc.max-connection}")
    int maxConnection,
    @Value("${my.datasource.etc.timeout}")
    Duration timeout,
    @Value("${my.datasource.etc.options}")
    List<String> options
    ) {
        return new MyDataSource(url, username, password, maxConnection, timeout, options);
    }
}
```

- 필드에서 사용할 수 있고 파라미터로도 사용할 수 있다.
- `@Value`가 타입 컨버팅을 해준다.
- 키를 찾지 못했을 경우 : 뒤에 기본값을 넣어줄 수 있다.
    - `@Value("${my.datasource.etc.max-connection:1}")`

## 외부설정 사용 - @ConfigurationProperties 시작

**Type-safe Configuration Properties**

- 외부 설정의 묶음 정보를 객체로 변환하느 기능을 제공. 타입 안전한 설정 속성
    - 객체를 사용하면 타입을 사용할 수 있다.
    - 잘못된 타입이 들어오는 문제를 방지할 수 있고 객체를 통해서 활용할 수 있는 부분들이 많아진다.
    - 외부 설정을 자바 코드로 관리하면서 그 정보 자체도 타입을 가지게 된다.
```java
@Data
@ConfigurationProperties("my.datasource")
public class MyDataSourcePropertiesV1 {
    private String url;
    private String username;
    private String password;

    private Etc etc = new Etc();

    @Data
    public static class Etc {
        private int maxConnection;
        private Duration timeout;
        private List<String> options;

    }
}
```

- 외부 설정을 주입 받을 객체를 생성한다. 각 필드를 외부 설정의 키 값에 맞춰 준비한다.
- @ConfigurationProperties는 외부 설정을 주입 받을 객체라는 뜻이다. key 묶음 시작점인 my.datasource를 명시해준다.
- 기본 주입 방식은 자바빈 프로퍼티 방식이다. getter, setter가 필요하다

- 타입에 맞지 않는 값이 들어오면 오류가 발생할 수 있다. 타입이 다르면 오류가 발생한다.
    - 그래서 타입 안정 속성이라고 한다.
    - `@ConfigurationProperties`로 만든 외부 데이터는 믿고 사용할 수 있다.
        - 파일뿐만이 아니라 environment 의 값을 다 읽어온다.

**표기법 변환**

- 스프링은 캐밥 표기법을 카멜 표기법으로 자동으로 바꿔준다.

`@EnableConfigurationProperties`

- 하나하나 properties 클래스를 등록하면 확장에 유연하지 못하다.

main 클래스에 `@ConfigurationPropertiesScan` 를 넣어주면 된다. 범위를 직접 정해줄 수도 있다.

### 문제

- 빈으로 등록한 properties는 setter로 사용해 값을 변경할 수가 있다.
- 외부 설정값은 초기에만 설정되고 변경되면 안될 것이다. 따라서 setter를 제거하는 방법을 사용하면 된다.
- 값을 아예변경하지 못하게 근본적으로 불변성을 보장해주자.


## 외부설정 사용 - @ConfigurationProperties 생성자

- `@ConfigurationProperties` 는 자바빈 프로퍼티 규약이 아닌 생성자로 만들 수 있는 기능을 제공한다

```java
@Getter
@ConfigurationProperties("my.datasource")
public class MyDataSourcePropertiesV2 {

    private String url;
    private String username;
    private String password;
    private Etc etc;

    public MyDataSourcePropertiesV2(String url, String username, String password, @DefaultValue Etc etc) {
        this.url = url;
        this.username = username;
        this.password = password;
        this.etc = etc;
    }

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
}
```
- 생성자를 통해 설정 정보가 주입된다.
- `@DefaultValue`를 사용하면 기본값이 주입이 된다.

@ContructorBinding

- 스프링 3.0 이전에는 생성자 바인딩 시에 @ConstructorBinding 애너테이션을 필수로 사용해야 했다.
- 스프링 3.0부터는 생성자가 하나일 때는 생략할 수 있다. 생성자가 둘 이상일 때는 생성자에 애너테이션을 적용해야 한다.

### 문제

- 타입은 맞는데 숫자의 범위가 기대하는 것과 다를 수 있지 않다.
    - max-connection은 최소 1 이상으로 설정해야 문제가 발생하지 않는다.
- 값을 검증해줄 필요가 있다.

## 외부설정 사용 - @ConfigurationProperties 검증

- 자바의 자바 빈 검증기(java bean validation)이라는 표준 검증기를 사용해서 검증을 해보자.
- @ConfigurationProperties 는 자바 객체이기 때문이 자바 빈 검증기를 사용할 수 있다.

```java
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

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

    public MyDataSourcePropertiesV3(String url, String username, String password, @DefaultValue Etc etc) {
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

        public Etc(int maxConnection, Duration timeout, @DefaultValue("DEFAULT") List<String> options) {
            this.maxConnection = maxConnection;
            this.timeout = timeout;
            this.options = options;
        }
    }
}
```

- `jakarta.validation` 로 시작하는 것은 자바 표준 검증기에서 지원하는 기능이다.
- `org.hibernate.validator.constraints` 로 시작하는 것은 아직 자바 표준 검증기에서 표준화된 기능은 아니고 하이버네이트 검증기라는 표준 검증기의 구현체에서 직접 제공하는 것이다.
- 하이버네이트 검증기는 크게 문제가되지는 않는다.

정리

- 가장 좋은 예외는 컴파일 예외이다.
- 그리고 애플리케이션 로딩 시점에 발생하는 예외도 좋다.
- 고객 서비스 중에 발생하는 런타임 예외가 가장 좋지 않다.

## YAML

스프링은 설정 데이터를 사용할 때 application.yml 형식도 지원한다.

YAML(YAML Ain’t Markup Langugage)은 사람이 읽기 좋은 데이터 구조를 목표로 한다.

- YAML의 가장 큰 특징은 사람이 읽기 좋게 계층 구조를 이룬다는 점이다.
- YAML은 공백으로 계층 구조를 만든다.
- 구본 기호로 `:` 를 사용한다.

스프링은 YAML의 계층 구조를 properties 처럼 평평하게 만들어서 읽어드린다.

> application.properties, application.yml 을 같이 사용하면 properties가 우선권을 가진다. 둘을 함께 사용하는 것은 일관성이 없으므로 권장하지 않는다. 실무에서는 yml를 선호한다.

# @Profile

---

- 각 환경마다 서로 다른 빈을 등록해야 한다면 어떻게 해야할까?
    - 결제 기능에서 로컬, 개발 환경에서는 가짜 결제 기능이 있는 스프링 빈을 등록하고 운영환경에서는 실제 결제 기능을 제공하는 스프링 빈을 등록한다.
    
    ```java
    @Slf4j
    @Configuration
    public class PayConfig {
    
        @Bean
        // profile 을 설정하지 않으면 default profile 이 활성화된다.
        @Profile("default")
        public LocalPayClient localPayClient() {
            log.info("LocalPayClient 빈 등록");
            return new LocalPayClient();
        }
    
        @Bean
        @Profile("prod")
        public ProdPayClient prodPayClient() {
            log.info("ProdPayClient 빈 등록");
            return new ProdPayClient();
        }
    }
    ```
    
    ```java
    @Component
    @RequiredArgsConstructor
    public class OrderRunner implements ApplicationRunner {
    
        private final OrderService orderService;
    
        // 스프링이 빈 초기화가 끝나고 로딩이 완료되는 시점에 실행시킨다.
        @Override
        public void run(ApplicationArguments args) throws Exception {
            orderService.order(1000);
        }
    }
    ```
    
    profile 은 특정 조건에 맞는 경우에 빈을 등록하는 것이다.
    
    ```java
    ...
    @Conditional(ProfileCondition.class)
    public @interface Profile {
    
    	/**
    	 * The set of profiles for which the annotated component should be registered.
    	 */
    	String[] value();
    
    }
    ```
    
    ```java
    class ProfileCondition implements Condition {
    
    	@Override
    	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    		MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
    		if (attrs != null) {
    			for (Object value : attrs.get("value")) {
    				if (context.getEnvironment().acceptsProfiles(Profiles.of((String[]) value))) {
    					return true;
    				}
    			}
    			return false;
    		}
    		return true;
    	}
    
    }
    ```
    
    condition을 구현한 구현체이다.
    
    
