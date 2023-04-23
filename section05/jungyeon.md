# 5. 자동 구성(Auto Configuration)
## 스프링 부트의 자동 구성    
- 스프링 부트는 자동 구성(Auto Configuration)이라는 기능을 제공하는데,    
  일반적으로 자주 사용하는 수많은 빈들을 자동으로 등록해주는 기능이다.       
- JdbcTemplate , DataSource , TransactionManager     
  모두 스프링 부트가 자동 구성을 제공해서 자동으로 스프링 빈으로 등록된다     
- @AutoConfiguration 에 자동 구성의 순서를 지정할 수 있다.
- @AutoConfiguration 도 설정 파일이다. 내부에 @Configuration 이 있는 것을 확인할 수 있다.
            
- 하지만 일반 스프링 설정과 라이프사이클이 다르기 때문에 컴포넌트 스캔의 대상이 되면 안된다 
- 그래서 스프링 부트가 제공하는 컴포넌트 스캔에서는 @AutoConfiguration 을 제외하는    
AutoConfigurationExcludeFilter 필터가 포함되어 있다.
```
@SpringBootApplication
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes =
TypeExcludeFilter.class),
@Filter(type = FilterType.CUSTOM, classes =
AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {...}
```
자동 구성이 내부에서 컴포넌트 스캔을 사용하면 안된다. 대신에 자동 구성 내부에서 @Import 는 사용할 수
있다.
## 자동 구성을 언제 사용하는가?
- AutoConfiguration 은 라이브러리를 만들어서 제공할 때 사용하고, 그 외에는 사용하는 일이 거의 없다. 
- 왜냐하면 보통 필요한 빈들을 컴포넌트 스캔하거나 직접 등록하기 때문이다. 하지만 라이브러리를 만들어서
제공할 때는 자동 구성이 유용하다. 실제로 다양한 외부 라이브러리들이 자동 구성을 함께 제공한다.
- 보통 이미 만들어진 라이브러리를 가져다 사용하지, 반대로 라이브러리를 만들어서 제공하는 경우는 매우
드물다. 그럼 자동 구성은 왜 알아두어야 할까?
- 자동 구성을 알아야 하는 진짜 이유는 개발을 진행 하다보면 사용하는 특정 빈들이 어떻게 등록된 것인지
확인이 필요할 때가 있다. 이럴 때 스프링 부트의 자동 구성 코드를 읽을 수 있어야 한다. 
- 자동화는 매우 편리한 기능이지만 자동화만 믿고 있다가 실무에서 문제가
발생했을 때는 파고 들어가서 문제를 확인하는 정도는 이해해야 한다. 이번에 학습한 정도면 자동 구성
코드를 읽는데 큰 어려움은 없을 것이다.
