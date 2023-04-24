# Section 5. 자동 구성(Auto Configuration)정리

- 회원 데이터를 보관하기 관리하기 위해서 빈으로 여러가지 객체 등록해서 귀찮다.

## 자동구성 확인

- `DbConfig`의  빈 등록을 하지 않아도 빈 등록 테스트가 정상 동작 → 스프링 부트가 자동으로 등록해준 것들

## 스프링 부트의 자동 등록

- spring-boot-autoconfigure > `JdbcTemplateAutoConfiguration`

    ```java
    @AutoConfiguration(after = DataSourceAutoConfiguration.class)
    @ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
    @ConditionalOnSingleCandidate(DataSource.class)
    @EnableConfigurationProperties(JdbcProperties.class)
    @Import({ DatabaseInitializationDependencyConfigurer.class, JdbcTemplateConfiguration.class, NamedParameterJdbcTemplateConfiguration.class })
    public class JdbcTemplateAutoConfiguration {} 
    ```

    - `@AutoConfiguration` : 자동 구성을 사용하기 위한 애노테이션
        - 내부에 `@Configuration`을 포함
        - `after` value를 통해서 순서 지정
    - `@ConditionalOnClass` : 빈 등록을 위해 충족해야 할 조건 제어하는 애노테이션
        - `value` 로 존재하는 클래스가 모두 있는 경우에만 등록한다.
    - `@Import` : 스프링에서 자바 설정을 추가할 때 사용한다.
- spring-boot-autoconfigure > `JdbcTemplateConfiguration`

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

    - `@ConditionalOnMissingBean` : 해당 클래스의 빈이 없는 경우에만 빈 등록 수행
        - `JdbcOperations` : `JdbcTemplate`의 조상 인터페이스로 실질적인 기능 인터페이스로 해당 인터페이스가 존재할 경우 다른 구현체가 있어 `JdbcTemplate`를 등록하지 않는 다.
          보통 개발자가 직접 빈으로 등록하면 해당 설정은 동작하지 않는다.

### Auto Configuration - 용어, 자동설정 or 자동 구성

- 자동 설정, 자동 구성의 두 용어로 번역 됨.

**자동 설정**

- 스프링이 동작하는 환경을 설정해 주기 때문에 자동 설정이라는 용어도 맞다.

**자동 구성**

- 필요한 빈들의 적절하게 선택하여 구성해주기 때문에 자동 구성이라는 용어도 맞다.

## @AutoConfiguration(자동 구성) 직접 만들기

**Memory 예제**

### **@Conditional:** 특정 조건일 때만 해당 기능이 활성화 되로록 하기

→ 같은 소스 코드이지만 특정 조건이 만족할 때 특정 빈을 등록하도록 도와줌.

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
	Class<? extends Condition>[] value();
}
```

- `Condition` 하위 타입의 조건들을 가지고 있음.

```java
@FunctionalInterface
public interface Condition {
	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```

- 전달된 `Context`와 조건 설정에 사용된 `metadata`를 이용해서 조건 충족 여부를 판단

> 하다가 생긴 의문 왜 3번 찍히지?
>
>
> ![image](https://user-images.githubusercontent.com/61923768/233367958-9d7be28c-f8c7-44ee-9ccc-6ea6686d6b99.png)
>
> 디버깅 해보니 `findCandidateComponent`, `invokdBeanFactoryPostProcessors`, `loadBeanDefinitionsForConfigurationClass` 총 3군데에서 호출함.
>
> 요약하면 후보 탐색 / 객체 생성 / 객체 초기화에서 호출 됨.
>

* 참고
  스프링은 외부 설정을 모두 추상화하여 `Enviroment`로 통합해서 처리한다.

### 스프링이 이미 만든 @Conditional 관련 클래스들

- `@ConditionalOnProperty`

    ```java
    class MemoryCondition : Condition {
        override fun matches(context: ConditionContext, metadata: AnnotatedTypeMetadata): Boolean {
            // -Dmemory=on
            val memory = context.environment.getProperty("memory")
            return memory.equals("ON", ignoreCase = true)
        }
    }
    ```

    ```java
    @Configuration
    @Conditional(value = [MemoryCondition::class])
    @ConditionalOnProperty(name = ["memory"], havingValue = "on")
    class MemoryConfig {
    
        @Bean
        fun memoryController(): MemoryController {
            return MemoryController(memoryFinder())
        }
    
        @Bean
        fun memoryFinder(): MemoryFinder {
            return MemoryFinder()
        }
    }
    ```

  `@ConditionalOnProperty(name = ["memory"]**,** havingValue = "on")`

  을 통해서 특정 이름의 프로퍼티가 특정 값을 가질 때 라는 조건을 손쉽게 만들 수 있음.

  대소문자는 구분하지 않으며 `missingIfValue`라는 boolean 값은 이 프로퍼티가 없을 때 Conditional의 결과를 의미하고 기본 값은 `false`


## 라이브러리 만들기

메모리 조회 기능을 라이브러리로 만들어보자

사용자가 사용하는 방법.

→ 메모리 조회를 순수 `jar`로 만들고 파일로 의존성 가지기

→ `config` 설정을 통해서 하나하나 `bean` 등록하기

위와 같은 경우 라이브러리 설정을 잘 읽어서 하나하나 등록해야하는 불편함이 있다.

이를 간단하게 해결해주는 것이 `@AutoConfiguration`이다.

→ 이를 스프링부트가 읽어서 사용하게 하기 위해서는 `resource/spring/org.springboot.autoconfigure.Autoconfiguration.imports` 라는 파일 안에 `@AutoConfiguration` 어노테이션이 붙은 클래스의 풀네임을 넣어주어야 한다.

### 스프링 부트의 자동 구성 실행 방식

`@SpringBootApplication` → `@EnableAutoConfiguration` → `@Import(AutoConfifurationImportSelector.class)`

### **ImportSelector class**

`@Import`에 설정 정보를 추가하는 2가지 방법

1. 정적인 방법: `@Import(class)` 코드에 박혀있어서 설정을 사용할 대상으로 변경할 수 없다.

    ```kotlin
    @Configuration
    @Import(value = [SampleConfig::class])
    class StaticImportConfig {...}
    ```

2. 동적인 방법: `@Import(ImportSelector)` 코드로 프로그래밍해서 사용할 대상을 동적으로 선택

    ```java
    public interface ImportSelector {
    	String[] selectImports(AnnotationMetadata importingClassMetadata);
    
    	@Nullable
    	default Predicate<String> getExclusionFilter() {
    		return null;
    	}
    }
    ```

    - `selectImports()`가 반환하는 Stirng 값과 풀네임이 같은 클래스 찾아서 등록한다.

   스프링 부트가 사용하는 `AutoConfigurationImportSelector.class` 의 `selectImports()`

    ```java
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
    	if (!isEnabled(annotationMetadata)) {
    		return NO_IMPORTS;
    	}
    	AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
    	return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
    ```

    1. `isEnabled`가 `false`면 import 하지 않음.
        - isEnabled 함수 코드

            ```java
            protected boolean isEnabled(AnnotationMetadata metadata) {
            	if (getClass() == AutoConfigurationImportSelector.class) {
            		return getEnvironment().getProperty(EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY, Boolean.class, true);
            	}
            	return true;
            }
            ```

        - `AutoConfigurationImportSelector` 클래스가 아니면 무조건 ture!
        - 환경 관련 프로퍼티 중 `EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY("spring.boot.enableautoconfiguration"`) 값을 반환.
        - 기본값이 true라 없으면 true!
    2. `autoConfigurationEntry`를 가져오는 과정 (`getAutoConfigurationEntry()`)

        ```java
        protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        	if (!isEnabled(annotationMetadata)) {
        		return EMPTY_ENTRY;
        	}
        	AnnotationAttributes attributes = getAttributes(annotationMetadata);
        	List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
        	configurations = removeDuplicates(configurations);
        	Set<String> exclusions = getExclusions(annotationMetadata, attributes);
        	checkExcludedClasses(configurations, exclusions);
        	configurations.removeAll(exclusions);
        	configurations = getConfigurationClassFilter().filter(configurations);
        	fireAutoConfigurationImportEvents(configurations, exclusions);
        	return new AutoConfigurationEntry(configurations, exclusions);
        }
        ```

        - `attribute`와 중복제거한 `configurations`을 가져오기
            - configurations 가져올 때 사전에 후보들을 가져옴(`getCondidateConfigurations()`)

                ```java
                protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
                	List<String> configurations = ImportCandidates.load(AutoConfiguration.class, getBeanClassLoader())
                		.getCandidates();
                	Assert.notEmpty(configurations,
                			"No auto configuration classes found in "
                					+ "META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports. If you "
                					+ "are using a custom packaging, make sure that file is correct.");
                	return configurations;
                }
                ```

            - 여기서 `ImportCondidates.load()`를 이용하는 데 내부에 LOCATION를 이용

                ```java
                public final class ImportCandidates implements Iterable<String> {
                	private static final String LOCATION = "META-INF/spring/%s.imports";
                	...
                }
                ```

            - 이때 이 `Location`이 아까 우리가 적은 파일
        - 제외해야하는 애들(`exculsions`) 정리 후 잘못된 제외 클래스 체크 후 제외
        - 필터링 후 이벤트 발생
        - 엔트리로 변환하여 반환
    3. 이걸 `String[]`로 변환해서 반환
