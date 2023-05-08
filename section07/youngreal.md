# 외부설정과 프로필2
###  Environment
```java
@Configuration
public class EnvConfig {

	private final Environment env;

	public EnvConfig(Environment env) {
		this.env = env;
	}

	@Bean
	public MyDatasource mydataSource() {
		String url = env.getProperty("my.datasource.url");
		String username = env.getProperty("my.datasource.username");

		return new MyDataSource(url, username);
	}
}
```
- Environment를 사용하면 해당값들을 읽을수있다. 스프링 내부변환기가  사용되어 타입도 변환될수있다.

- Environment에서 값을 계속 꺼내야한다는 귀찮은 단점이있다.

### @Value

- 필드, 파라미터에 사용해 주입할수있다.
- 주입받아야하는 값이 한두개가 아닌 여러개라면 조금 번거로울수있다.

### @ConfigurationProperties
```java
@ConfigurationProperties("my.datasource")
public class MyDataSourceProperties {
	
	private String url;
	private String username;
	private String password;
	private Etc etc;
	
	@Data
	public static class Etc {
		private int maxConnection;
		private Duration timeout;
		private List<String> options = new ArrayList<>();
	}
}
```

```java
@EnableConfigurationProperties(MyDataSourceProperties.class) // 속성을 주입하고 스프링빈으로 등록
public class MyDataSourceConfigV1 {
	
	private final MyDataSourceProperties properties;

	public MyDataSourceCOnfigV1(MyDataSourceProperties properties) {
		this.properties = properties;
	}

	@Bean
	public MyDataSource myDataSource() {
		return new MyDataSource(
			properties.getUrl(),
			properties.getUrl()
		)
	}
}

```

- 외부설정의 묶음정보를 객체로 변환하는 기능을 제공한다.

- 지정해둔 타입이외의 타입이 들어오면 예외가 발생하기때문에 타입안전하게 값을 세팅할수있다.

- main apllication에 @ConfigurationPropertiesScan 을 사용하면 이미 등록한 빈 클래스를 한번더 설정해야하는 번거로움을 피할수있다.

- setter를 두지않고 **생성자로도 값을 주입할수있다**

- java validation으로 **검증도 가능하다**

- 외부설정을 객체로 관리해 **타입안전**하다
