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

## `@Conditional`

- 메모리 조회 기능이 특정 조건일 때만 해당 기능이 활성화 되도록
- 예를 들어 개발 서버에서만 사용할 수 있도록
- 핵심은 소스코드를 고치지 않고 이런 것이 가능해야 한다.
- 같은 소스코드인데 특정 상황일 때만 특정 빈들을 등록해서 사용하도록 도와주는 기능이 바로 @Conditional이다.

Condition

```java
@FunctionalInterface
public interface Condition {
	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);

}
```

- `matches()` 메서드가 true를 반환하면 조건에 만족해서 동작하고 false를 반환하면 동작하지 않는다.
- `ConditionContext`: 스프링 컨테이너, 환경 정보등을 담고 있다.
- `AnnotatedTypeMetadata`: 애노테이션 메타 정보를 담고 있다.

## @Conditional - 다양한 기능

`@ConditionalOnProperty(name = "memory", havingValue = "on")`

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.TYPE, ElementType.METHOD })
@Documented
@Conditional(OnPropertyCondition.class)
public @interface ConditionalOnProperty {
```

`ConditionalOnProperty`는 `Conditional`을 사용해서 property를 확인하는 조건 분기가 들어가있다.

## @ConditionalOnXxx

스프링은 @Conditional 과 관련해서 개발자가 편리하게 사용할 수 있도록 많은 @ConditionalOnXxx를 제공한다.

- @ConditionalOnClass, @ConditionalOnMissingClass
    - 클래스가 있는 경우는 동작한다. missing은 반대
- @ConditionalOnBean, @ConditionalOnMissingBean
    - 빈이 등록되어 있는 경우 동작
- @ConditionalOnProperty
    - 환경 정보가 있는 경우 동작
- @ConditionalOnResource
    - 리소스가 있는 경우 동작
- @ConditionalOnWebApplication, @ConditionalOnNotWebApplication
    - 웹 애플리케이션인 경우 동작
- @ConditionalOnExpression
    - SpEL 표현식에 만족하는 경우 동작

> ConditionalOnXxx 공식 메뉴얼
[https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration.condition-annotations](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration.condition-annotations)
> 

ConditionalOnXxx 는 스프링 부트 자동 구성에 사용된다.

- @Conditional은 스프링 프레임워크의 기술인데 이를 확장에서 스프링부트가 사용한다.
- @AutoConfiguration: 자동 구성이 어떻게 동작하는지 내부 원리 이해

### 자동 구성 대상 지정

- 스프링 부트 자동구성을 적용하려면 다음 파일에 자동 구성 대상을 꼭 지정해주어야 한다.
- 폴더 위치와 파일 이름이 길기 때문에 주의하자.

`org.springframework.boot.autoconfigure.AutoConfiguration.imports`

![image](https://user-images.githubusercontent.com/66561524/233795409-34aba42a-0e0d-4c20-84ec-2d2ab05074ae.png)


## 자동 구성 이해1 - 스프링 부트의 동작

스프링부트는 다음 경로에 있는 파일을 읽어서 스프링 부트 자동 구성으로 사용

`resoureces/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

스프링 부트 자동 구성이 동작하는 원리는 다음과 같다.

`@SpringBootApplication` → `@EnableAutoConfiguration` → `@Import(AutoConfigurationImportSelector.class)`

```java
@SpringBootApplication
public class ProjectV1Application {

    public static void main(String[] args) {
        SpringApplication.run(ProjectV1Application.class, args);
    }

}
```

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelecto그ㄴ데 
```

- `@Import`는 주로 스프링의 설정 정보(주로 `@Configuration`)을 사용
- `@AutoConfigurationImportSelector` 는 `@Configuration`이 아니다.

`ImportSelector`를 알아야 한다.

## 자동 구성 이해2 - ImportSelector

`@Import` 에 설정 정보를 추가하는 방법은 2가지가 있다.

- 정적인 방법: `@Import` (클래스) 이것은 정적이다. 코드에 대상이 딱 박혀있다. 설정으로 사용할 대상을 동적으로 변경할 수 없다.

```java
@Configuration
@Import({AConfig.class, BConfig.class})
public class AppConfg {..}
```

- 동적인 방법: `@Import(ImportSelector)`코드로 프로그래밍해서 설정으로 사용할 대상을 동적으로 선택할 수 있다.

```java
public interface ImportSelector {

			String[] selectImports(AnnotationMetadata importingClassMetadata);
```

- `String[]` 에 설정정보를 반환하면 된다.

## @EnableAutoConfiguration

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

- AutoConfigurationImportSelector 는 ImportSelector 의 구현체이다. 따라서 설정 정보를 동적으로 선택할 수 있다.
- `resoureces/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

정리

- 스프링 부트의 자동 구성을 직접 만들어서 사용할 때는 다음을 참고하자.
- @AutoConfiguration 에 자동 구성의 순서를 지정할 수 있다.
- @AutoConfiguration도 설정 파일이다. 내부에 @Configuration 이 있는 것을 확인할 수 있다.
    - 하지만 일반 스프링 설정과 라이프사이클이 다르기 때문에 컴포넌트 스캔의 대상이 되면 안된다.
- 파일에 지정해서 사용해야 한다.
- `resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`
- 그래서 스프링 부트가 제공하는 컴포넌트 스캔에서는 `@AutoConfiguration` 을 제외하는`AutoConfigurationExcludeFilter` 필터가 포함되어 있다.

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { 
	@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
	@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) 
}) 
public @interface SpringBootApplication {...}
```

자동 구성이 내부에서 컴포넌트 스캔을 사용하면 안된다. 대신에 자동 구성 내부에서 @Import는 사용할 수 있다.

**자동 구성을 언제 사용하는가?**

- 자동구성은 라이브러리를 만들어서 제공할 때 사용하고, 그 외에는 사용하는 일이 거의 없다. 
- 보통 필요한 빈들을 컴포넌트 스캔하거나 직접 등록하기 때문이다. 
- 하지만 라이브러리를 만들어서 제공할 때는 자동 구성이 유용하다. 실제로 다양한 외부 라이브러리들이 자동 구성을 함께 제공한다.

보통 이미 만들어진 라이브러리를 가져다 사용하지, 반대로 라이브러리를 만들어서 제공하는 경우는 매우 드물다. 그럼 자동 구성은 왜 알아두어야 할까?

- 자동 구성을 알아야 하는 진짜 이유는 개발을 진행 하다보면 사용하는 특정 빈들이 어떻게 등록된 것인지 확인이 필요할 때가 있다. 
- 이럴 때 스프링 부트의 자동 구성 코드를 읽을 수 있어야 한다. 그래야 문제가 발생했을 때 대처가 가능하다.
- 자동화는 매우 편리한 기능이지만 자동화만 믿고 있다가 실무에서 문제가 발생했을 때는 파고 들어가서 문제를 확인하는 정도는 이해해야 한다. 이번에 학습한 정도면 자동 구성 코드를 읽는데 큰 어려움은 없을 것이다.


