# section 05 자동구성(Auto Configuration)

## 스프링 부트의 자동 구성

- 스프링 부트는 자동 구성(Auto Configuration)이라는 기능을 제공하는데 일반적으로 자주 사용하는 수많은 빈들을 자동으로 등록해주는 기능이다.
- 스프링 부트 스타터의 의존성에는 autoconfigure가 들어가있다.

![image](https://user-images.githubusercontent.com/66561524/233786974-dfa6470b-c7f0-423d-8d57-707d3ab89e9a.png)

- 수많은 자동 구성이 들어가있다.

![image](https://user-images.githubusercontent.com/66561524/233787046-63ae7eae-fb59-45e2-9d64-4c82ffda2a4e.png)

```java
@AutoConfiguration(after = DataSourceAutoConfiguration.class)
@ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
@ConditionalOnSingleCandidate(DataSource.class)
@EnableConfigurationProperties(JdbcProperties.class)
@Import({ DatabaseInitializationDependencyConfigurer.class, JdbcTemplateConfiguration.class,
		NamedParameterJdbcTemplateConfiguration.class })
public class JdbcTemplateAutoConfiguration {

}
```

- `AutoConfiguration`: 자동구성을 사용하려면 이 애노테이션을 등록해야 한다.
    - 자동구성도 내부에 `@Configuration` 이 있어서 빈을 등록하는 자바 설정 파일로 사용할 수 있다.
    - `after = DataSourceAutoConfiguration.class`
        - 자동구성이 실행되는 순서를 지정할 수 있다. `JdbcTemplate`은 `DataSource`가 필요하기 때문에 `DataSource`를 자동으로 등록해주는 `DataSourceAutoConfiguration` 다음에 실행하도록 설정되어있다.
- `@ConditionalOnClass({ DataSource.class, JdbcTemplate.class })`
    - IF문과 유사한 기능을 제공한다. 이런 클래스가 있는 겨웅에만 설정이 동작한다. 만약 없으면 여기 있는 설정들이 모두 무효화 되고 빈도 등록되지 않는다.
    - `@ConditionalXxx` 시리즈가 있다.
    - `JdbcTemplate`은 `DataSource`, `JdbcTempalte`라는 클래스가 있어야 동작할 수 있다.
- `@Import`: 스프링에서 자바 설정을 추가할 때 사용한다.

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnMissingBean(JdbcOperations.class)
class JdbcTemplateConfiguration {

	@Bean
	@Primary
	JdbcTemplate jdbcTemplate(DataSource dataSource, JdbcProperties properties) {
		JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
		JdbcProperties.Template template = properties.getTemplate();
		jdbcTemplate.setFetchSize(template.getFetchSize());
		jdbcTemplate.setMaxRows(template.getMaxRows());
		if (template.getQueryTimeout() != null) {
			jdbcTemplate.setQueryTimeout((int) template.getQueryTimeout().getSeconds());
		}
		return jdbcTemplate;
	}

}
```
- `@ConditionalOnMissingBean(JdbcOperations.class)`
    - JdbcOperations 빈이 없으면 동작
        
      ![image](https://user-images.githubusercontent.com/66561524/233787065-efaf1e82-53ea-4698-bca7-cd5f450f9ed6.png)
        
    - `JdbcTemplate`의 부모 인터페이스가 `JdbcOperations` 이다.
    - `JdbcTemplate` 이 빈으로 등록되어있지 않은 경우에만 동작
    - 만약 이런 기능이 없으면 내가 등록한 `JdbcTemplate`과 자동 구성이 등록하는 `JdbcTemplate`이 중복 등록되는 문제가 발생
    - 개발자가 직접 빈을 등록하면 개발자가 등록한 빈을 사용하고 자동 구성은 동작하지 않는다.
- `JdbcTempalte`이 몇가지 설정을 거쳐서 빈으로 등록되는 것을 확인

자동 등록 설정

- `JdbcTempalteAutoConfiguration`: `JdbcTemplate`
- `DataSourceAutoConfiguration`: `DataSource`
- `DataSourceTransactionManagerAutoConfiguration`: `TransactionManager`

스프링 부트가 제공하는 자동 구성

[https://docs.spring.io/spring-boot/docs/current/reference/html/auto-configuration-classes.html](https://docs.spring.io/spring-boot/docs/current/reference/html/auto-configuration-classes.html)

- 스프링 부트 프로젝트를 사용하면 자동으로 설정이 등록된다.

Auto Configuration - 용어, 자동 설정? 자동 구성

자동 등록 설정

- `JdbcTempalteAutoConfiguration`: `JdbcTemplate`
- `DataSourceAutoConfiguration`: `DataSource`
- `DataSourceTransactionManagerAutoConfiguration`: `TransactionManager`

스프링 부트가 제공하는 자동 구성

[https://docs.spring.io/spring-boot/docs/current/reference/html/auto-configuration-classes.html](https://docs.spring.io/spring-boot/docs/current/reference/html/auto-configuration-classes.html)

- 스프링 부트 프로젝트를 사용하면 자동으로 설정이 등록된다.

Auto Configuration - 용어, 자동 설정? 자동 구성

자동 설정

빈들을 자동으로 등록해서 스프링이 동작하는 환경을 자동으로 설정해준다.

자동 구성

스프링 실행에 필요한 빈들을 적절하게 배치해야 한다. 자동 설정, 자동 구성 두 용어 모두 맞는 말이다. 자동 설정은 넓게 사용되는 의미이고 자동 구성은 실행에 필요한 컴포넌트 조각을 자동으로 배치한다는 좁은 의미에 가깝다.

- Auto Configuration은 자동 구성이라는 단어를 주로 사용하고 문맥에 따라서 자동 설정이라는 단어도 사용하겠다.
- Configuration이 단독으로 사용될 때는 설정이라는 단어를 사용한다.

스프링 부트가 제공하는 자동 구성 기능을 이해하려면 두 가지 개념을 이해해야 한다.

- `@Conditional`: 특정 조건에 맞을 때 설정이 동작하도록 한다.
- `@AutoConfiguration`: 자동 구성이 어떻게 동작하는지 내부 원리 이해
