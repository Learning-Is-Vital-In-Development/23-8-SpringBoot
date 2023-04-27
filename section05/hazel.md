# 5. 자동 구성 (Auto Configuration )

스프링 부트는 `자동구성(Auto Configuration)`이라는 기능을 제공하는데, 일반적으로 자주 사용하는 수많은 빈들을 자동으로 등록해주는 기능이다.
이런 자동 구성 덕분에 개발자는 반복적이고 복잡한 빈 등록과 설정을 최소화하고 개발을 빠르게 시작할 수 있다.

- 부트에서 제공하는 auto-configuration 목록
: [Auto-configuration Classes](https://docs.spring.io/spring-boot/docs/current/reference/html/auto-configuration-classes.html)
  
### 자동설정/ 자동구성 용어

- ***자동 설정***
  : Configuration 이라는 단어가 컴퓨터 용어에서는 환경 설정, 설정이라는 뜻으로 자주 사용된다. Auto Configuration은 크게 보면 빈들을 자동으로 등록해서 스프링이 동작하는 환경을 자동으로 설정해주기 때문에 자동 설정이라는 용어도 맞다.

- ***자동 구성***
  : Configuration 이라는 단어는 구성, 배치라는 뜻도 있다.
  예를 들어서 컴퓨터라고 하면 CPU, 메모리등을 배치해야 컴퓨터가 동작한다. 이렇게 배치하는 것을 구성이라 한다. 스프링도 스프링 실행에 필요한 빈들을 적절하게 배치해야 한다. 자동 구성은 스프링 실행에 필요한 빈들을 자동으로 배치해주는 것이다.

즉 , 두 용어 모두 맞는 말이고 자동설정이 넓게 사용되는 의미이다.


그리고 이 두가지 개념을 이해해야한다.

- `@Conditional` : 특정 조건에 맞을 때 설정이 동작하도록한다. 
- `@AutoConfiguration` : 자동 구성이 어떻게 동작하는지 내부 원리 이해

# @Conditional
같은 소스 코드인데 특정 상황일 때만 특정 빈들을 등록해서 사용하도록 도와주는 기능이 바로 `@Conditional`이다.

예를 들면 개발 서버에서 확인 용도로만 해당 기능을 사용하고, 운영 서버에서는 해당기능을 사용하지 않아야 할때, (빌드한 같은 jar파일을 개발서버에도 배포하고 , 운영서버에서 배포할때) 같은 소스 코드인데 특정 상황일때만 특정 빈들을 등록해서 사용하도록 도와주는 기능이 바로 `@Conditional`이다.



이 기능을 사용하려면 먼저 Condition 인터페이스를 구현해야 한다.

```java
package org.springframework.context.annotation;
public interface Condition {

	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);

}
```

그리고 이 인터페이스를 구현한 MemoryCondition가 존재할때, 

```java
//MemoryConfig

@Configuration
@Conditional(MemoryCondition.class) //추가 -!!
public class MemoryConfig {
    @Bean
    public MemoryController memoryController() {
        return new MemoryController(memoryFinder());
    }

    @Bean
    public MemoryFinder memoryFinder() {
        return new MemoryFinder();
    }
}
```

MemoryConfig의 적용 여부는 Conditional(MemoryCondition.class) 의 여부에 따라 달라진다. MemoryCondition의 match()를 실행하고  반환결과가 true가 될때, 해당 config가 등록이 되고, false가 되면 이 config를 실행하지 않고 bean이 등록되지 않는다.

# @Conditional의 다양한 기능
이렇듯 스프링은 `@Conditional` 관련하게 개발자가 편리하게 사용할 수 있도록 제공하는 수많은 `@ConditionalOnXxx`을 제공한다.

- `@ConditionalOnClass` , `@ConditionalOnMissingClass`: 클래스가 있는 경우 동작한다. 나머지는 그 반대
- `@ConditionalOnBean` , `@ConditionalOnMissingBean` :빈이 등록되어 있는 경우 동작한다. 나머지는 그 반대
- `@ConditionalOnProperty` :환경 정보가 있는 경우 동작한다.
- `@ConditionalOnResource` : 리소스가 있는 경우 동작한다.
- `@ConditionalOnWebApplication ,@ConditionalOnNotWebApplication`: 웹 애플리케이션인 경우 동작한다.
- `@ConditionalOnExpression` : SpEL 표현식에 만족하는 경우 동작한다.

# 자동구성의 이해 
스프링 부트는 다음 경로에 있는 파일을 모두 읽어서 스프링 부트 자동 구성으로 사용한다. 
resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports

그리고 다음과 같은 순서로 자동 구성은 동작된다.
1. .`@SpringBootApplication`
2. `@EnableAutoConfiguration` (auto configutation을 활성화해주는 기능)
3. `@Import(AutoConfigurationImportSelector.class)`

@Import 는 주로 스프링 설정 정보( @Configuration )를 포함할 때 사용한다.
그런데 AutoConfigurationImportSelector 를 열어보면 @Configuration 이 아니다. 이 기능을 이해하려면 ImportSelector 에 대해 알아야 한다.


# 자동 구성의 이해 - ImportSelector

@Import 에 설정 정보를 추가하는 방법은 2가지가 존재한다.

- **정적인 방법**: @Import (클래스) 이것은 정적이다. 코드에 대상이 딱 박혀 있다. 설정으로 사용할 대상을 동적으로 변경할 수 없다.
- **동적인 방법**: @Import ( ImportSelector ) 코드로 프로그래밍해서 설정으로 사용할 대상을 동적으로 선택할 수 있다. 즉 ImportSelector라는 인터페이스를 제공한다.

다시 @Import(AutoConfigurationImportSelector.class)를 보면  AutoConfigurationImportSelector 는 ImportSelector 의 구현체이다. 따라서 설정 정보를 동적으로 선택할 수 있다.
그리고  이 코드는 모든 라이브러리에 있는 다음 경로의 파일을 확인한다.  META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports


따라서 정리하면 스프링 부트 자동 구성이 동작하는 방식을 정리하면,

1.  `@SpringBootApplication`
2. `@EnableAutoConfiguration`
3. `@Import(AutoConfigurationImportSelector.class)`
4. 이제 `ImportSelector`가 `resources/META-INF/spring/
   org.springframework.boot.autoconfigure.AutoConfiguration.imports` 파일을 열어서 설정 정보 선택한다.
5. 해당 파일의 설정 정보가 스프링 컨테이너에 등록되고 사용된다.


### 자동구성을 언제 사용하는가?
AutoConfiguration 은 라이브러리를 만들어서 제공할 때 사용하고, 그 외에는 사용하는 일이 거의 없다.
특정 빈들이 어떻게 등록된 것인지
확인이 필요할 때가 있다. 이럴 때 스프링 부트의 자동 구성 코드를 읽을 수 있어야 한다. 그래야 문제가
발생했을 때 대처가 가능하다. 