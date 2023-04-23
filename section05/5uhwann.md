# # Section5: 자동 구성(Auto Configuration)

## 스프링 부트 자동 구성

스프링 부트는 일반적으로 자주 사용하는 수 많은 빈들을 자동으로 등록해주는 Auto Configuration이라는 기능을 제공한다.

이 Auto Configuration 덕분에 개발자는 반복적이고 복잡한 빈 등록과 설정을 최소화 할 수 있다.

**JdbcTemplateAutoConfiguration**

```java
package org.springframework.boot.autoconfigure.jdbc;

import javax.sql.DataSource;

import org.springframework.boot.autoconfigure.AutoConfiguration;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnSingleCandidate;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.boot.sql.init.dependency.DatabaseInitializationDependencyConfigurer;
import org.springframework.context.annotation.Import;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;

@AutoConfiguration(after = DataSourceAutoConfiguration.class)
@ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
@ConditionalOnSingleCandidate(DataSource.class)
@EnableConfigurationProperties(JdbcProperties.class)
@Import({ DatabaseInitializationDependencyConfigurer.class,
		JdbcTemplateConfiguration.class,
    NamedParameterJdbcTemplateConfiguration.class })
public class JdbcTemplateAutoConfiguration {

}
```

- `@AutoConfiguration` : 자동 구성을 사용하려면 이 애노테이션을 등록해야 한다.
    - `@AutoConfiguration` 내부에 `@Configuration` 이 있어서 빈을 등록하는 자바 설정 파일로 사용할 수 있다.
    - `after = DataSourceAutoConfiguration.class`
        - 자동 구성이 실행되는 순서를 지정할 수 있다. `JdbcTemplate` 은 `DataSource` 가 필요하기때문에 `DataSource` 를 자동으로 등록해주는`DataSourceAutoConfiguration` 다음에실행하도록 설정되어 있다.
- `@ConditionalOnClass({ DataSource.class, JdbcTemplate.class })`
    - if문과 유사한 기능을 제공. 해당 클래스가 있는 경우에만 설정이 동작. 만약 없으면 여기 있는설정들이 모두 무효화 되고, 빈도 등록되지 않는다.
- `@Import` : 스프링에서 자바 설정을 추가할 때 사용한다.

### Auto Configuration 용어

**자동 설정**

Auto Configuration은 크게 보면 빈들을 자동으로 등록해서 스프링이 동작하는 환경을 자동으로 설정해주기 때문에 자동 설정이라는 용어도 맞다.

**자동 구성**

자동 구성은 스프링 실행에 필요한 빈들을 자동으로 배치해주는 것이다.

→ 자동 설정은 넓게 사용되는 의미이고, 자동 구성은 실행에 필요한 컴포넌트 조각을 자동으로 배치한다는 더 좁은 의미에 가깝다.

## `@Conditional`

설정 파일을 항상 사용하는 것이 아닌 특정 조건일 때만 활성화 되도록 하고 싶을 때가 있다.

→ 개발 서버에서 확인 용도로만 사용하고, 운영 서버에서는 해당 설정을 사용하지 않는 것이다.

- 이때 핵심은 소스코드를 고치지 않고 이것이 가능해야 한다.
    - 프로젝트 빌드 파일 하나를 개발 서버, 운영 서버 모두에 배포

같은 소스 코드를 특정 상황일 때만 특정 빈들을 등록해서 사용하도록 하게 하는 기능이 `@Conditioanal`이다.

### `@Conditional` 사용

우선 `Condtion` 인터페이스를 구현해야 한다.

**Condition**

```java
 package org.springframework.context.annotation;

 public interface Condition {
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}

```

- `matches()` 메서드가 `true` 를 반환하면 조건에 만족해서 동작하고, `false` 를 반환하면 동작하지 않는다.
- `ConditionContext` : 스프링 컨테이너, 환경 정보등을 담고 있다.

설정 파일에 `@Conditional(Condtion구현 클래스)` 어노테이션 적용함으로써 `@Conditional`을 사용할 수 있다.

### `@Conditional` - 다양한 기능

**`@ConditionalOnXxx`**

스프링은 @Conditional 과 관련해서 개발자가 편리하게 사용할 수 있도록 수 많은`@ConditionalOnXxx`를 제공한다.

- `@ConditionalOnClass` , `@ConditionalOnMissingClass`
    - 클래스가 있는 경우 동작한다. 나머지는 그 반대
- `@ConditionalOnBean` , `@ConditionalOnMissingBean`
    - 빈이 등록되어 있는 경우 동작한다. 나머지는 그 반대
- `@ConditionalOnProperty`
    - 환경 정보가 있는 경우 동작한다.
- `@ConditionalOnResource`
    - 리소스가 있는 경우 동작한다.
- `@ConditionalOnWebApplication` , `@ConditionalOnNotWebApplication`
    - 웹 애플리케이션인 경우 동작한다.
- `@ConditionalOnExpression`
    - **SpEL**(****Spring Expression Language)**** 표현식에 만족하는 경우 동작한다.

## 자동 구성 이해 - 스프링 부트 동작

스프링 부트는 다음 경로의 파일을 읽어 스프링 부트 자동 구성으로 사용한다.

`resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

**스프링 부트 자동 구성 동작 순서**

`@SpringBootApplication` → `@EnableAutoConfiguration` → `@Import(AutoConfigurationImportSelector.class)`

- `@Import` 는 주로 스프링 설정 정보(`@Configuration`)를 포합할 때 사용하지만 `AutoConfigurationImportSelector`는 `@Configuration`이 아니다.

이를 이해하기 위해 ImportSelector에 대해 알아보자.

## 자동구성 이해 - ImportSelector

`**@Import` 에 설정 정보를 추가하는 2가지 방법**

- 정적인 방법: `@Import` (클래스) 이것은 정적이다.  설정으로 사용할 대상을 동적으로 변경할 수 없다.
- 동적인 방법: `@Import ( ImportSelector )` 코드로 프로그래밍해서 설정으로 사용할 대상을 동적으로 선택할 수 있다.

동적인 방법을 사용하기 위해서는 `ImportSelector` 인터페이스를 구현해야 한다.

**ImportSelector**

```java
 package org.springframework.context.annotation;
 public interface ImportSelector {
	 String[] selectImports(AnnotationMetadata importingClassMetadata);
		//...
}

```

- 설정 정보로 사용할 클래스를 동적으로 프로그래밍 하여 반환하면 된다.

`@EnableAutoConfuguration`의 `@Import(AutoConfigurationImportSelector.class)` 에서 `AutoConfigurationImportSelector.class`는 `ImportSelector`의 구현체이다. 따라서 설정 정보를 동적으로 선택 가능하다.

덕분에 모든 라이브러리 `resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 경로의 파일을 확인한다.

**정리**

스프링 부트 자동 구성이 동작하는 방식은 다음 순서로 확인 가능

- `@SpringBootApplication` → `@EnableAutoConfiguration` → `@Import(AutoConfigurationImportSelector.class)` → `resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 파일의 설정 정보 선택
- 해당 파일의 설정 정보 스프링 컨테이너 등록

## 자동 구성을 알아야 하는 이유

자동 구성을 알아야 하는 진짜 이유는 개발을 진행 하다보면 사용하는 특정 빈들이 어떻게 등록된 것인지 확인이 필요할 때가 있다.

이럴 때 스프링 부트의 자동 구성 코드를 읽을 수 있어야 한다.

그래야 문제가 발생했을 때 대처가 가능하다.
