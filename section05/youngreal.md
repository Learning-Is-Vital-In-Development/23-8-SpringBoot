## 스프링부트의 자동구성

- @AutoConfiguration : 자동구성을 이용하기위한 애노테이션, 순서도 지정할수있다.

- @ConditionalOnClass({A.class, B,class}): A라는 클래스와 B라는 클래스가 있는경우에만 설정한다.

### 자동 설정과 자동구성(Auto Configuration)

- 자동 설정: Configuration 이라는 단어가 설정, 환경설정이라는 뜻으로 자주 사용되므로 빈들을 자동등록해줘서 환경을 설정해주기때문에 자동설정이라는 용어도 적절하다

- 자동 구성: Configuration이 구성,배치라는 의미도 있는데 스프링은 스프링 실행에 필요한 빈들을 적절하게 배치하기때문에 적절하다.

자동설정은 좀더 포괄적인 의미이고, 자동구성은 실행에 필요한 컴포넌트들을 자동으로 배치한다는 좁은의미에 가깝다.


## @Conditional

특정 조건에 맞을때(if문) 설정을 동작시킨다. 
![](https://velog.velcdn.com/images/rodlsdyd/post/c27142de-c2c5-449c-a0e6-9aaa20ca16d3/image.png)

- JdbcTemplate이 빈으로 등록되어있지않으면 설정한다.
- 사용자가 특별히 빈으로 등록하면 그것이 빈으로 등록되고, 아닌경우 jdbcTeamplate을 등록한다.


## 자동구성 라이브러리 사용하기

1. 외부 라이브러리 직접만들고 jar파일을 프로젝트에 주입시키기

2. **프로젝트를 개발하는 입장에서 외부 라이브러리 어떤빈을 등록해야하는지 알아야하며, 등록도 해줘야한다.** (라이브러리 제작자가 문서화를 철저히 해야한다)

3. **@AutoConfiguration 을 설정한다면 사용하는입장에서 특별히 빈으로 등록해주지않아도 된다.**
  - 스프링부트가 라이브러리의 폴더를 찾아서 Autoconfiguration 대상을 인지하고 해당 파일의 세팅을 보고 빈 등록한다.
  

## 스프링 부트의 동작

#### 1. @SpringBootApplication에는 @EnableAutoConfiguration이 있다.

![](https://velog.velcdn.com/images/rodlsdyd/post/9b94b81e-f1e7-4dbb-99d5-00172f4234e7/image.png)

#### 2. @EnableAutoConfiguration은 스프링부트의 Auto Configuration을 활성화한다)
- META-INF/spring/org.springframework.boot.autoconfigure.Autoconfiguration.import에 정의된 Configuration들을 읽어서 등록한다.

- 주의할점은 @ComponentScan으로 먼저 빈을 읽고 난후 import에 정의된 Configuration들을 읽는다. 

- 해당 메타파일에 등록되어있는 클래스들도 전부 @Configuration이 붙어있으며, 이 클래스들에 특정조건이 붙어있을수있다
-> @Conditional이라던지..

> 스프링부트의 자동설정을 모아놓던 파일이 spring.factories 였던걸로 알고있었는데 강의에서는 경로가 바뀌었다.   찾아보니 스프링부트 2.7에서 변경되었다고한다. 
2.7 미만의 버전이라면 spring.factories에 담겨있을것이다.
  
  
## ImportSelector
@Import 에 프로그래밍해서 동적으로 설정정보를 추가하는방법

- 정적인 정보는 @Import({A.class, B.class})와같이 설정한다.

- META-INF/spring/org.springframework.boot.autoconfigure.Autoconfiguration.imports 파일의 클래스 정보를 읽고 클래스 내용에 따라 설정정보를 동적으로 선택할수있다.


### 자동구성의 사용

- AutoConfiguration은 결국 편의성을위해 라이브러리를 만들어서 제공할때 편의성을 주기위함이다. 사용자는 빈을 등록해주는 클래스를 만들고, 빈을 등록할 필요가없다.

- 라이브러리를 만들지 않더라도 자동으로 등록되는 특정 빈들을 확인해야할필요가 있을수있다.  어떤빈들이 어떻게 등록되었는지 확인할상태는 되어야한다. 



