# 8. 액츄에이터

## 프로덕션 준비 기능이란?

개발자가 애플리케이션을 개발할 때 기능 요구사항만 개발하는 것은 아니다. 서비스를 실제 운영 단계에 올리게 되면 개발자들이 해야하는 또 다른 중요한 업무가 있다. 바로 서비스에 문제가 없는지 모니터링하고 지표들을 심어서 감시하는 활동들이다.
운영 환경에서 서비스할 때 필요한 이런 기능들을 프로덕션 준비 기능이라 한다. 

- 지표(metric), 추적(trace), 감사(auditing)
- 모니터링

좀 더 구제적으로 설명하자면, 애플리케이션이 현재 살아있는지, 로그 정보는 정상 설정 되었는지, 커넥션 풀은 얼마나 사용되고 있는지 등을 확인할 수 있어야 한다.
스프링 부트가 제공하는 액추에이터는 이런 프로덕션 준비 기능을 매우 편리하게 사용할 수 있는 다양한 편의 기능들을 제공한다

# 액츄에이터

먼저 액츄에이터 라이브러리를 추가해야한다. 

```java
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

그리고 [http://localhost:8080/actuator](http://localhost:8080/actuator) 실행하면 확인 할수 있다. 

```yaml
management:
	endpoints:
		web:
			exposure:
				include: "*" //모든 엔드포인트를 웹에 노출
```

그리고 액츄에이터가 제공하는 수 많은 기능을 확인 할 수 있는데 이 기능 하나하나를 엔드포인트라고 한다. `health`는 헬스정보를, `beans`는 스프링 컨테이너에 등록된 빈을 보여준다. 

- [http://localhost:8080/actuator/health](http://localhost:8080/actuator/health) : 애플리케이션 헬스 정보를 보여준다.
- [http://localhost:8080/actuator/beans](http://localhost:8080/actuator/beans) : 스프링 컨테이너에 등록된 빈을 보여준다.

## 엔드포인트 설정

엔드포인트를 사용하려면 2가지의 과정이 필요하다.

**1. 엔드포인트 활성화 :**  활성화는 기능을 사용할지 말지  on,off 선택하는 것을 의미한다. 

**2. 엔드포인트 노출** : 노출은 활성화된 엔드포인트를 http에 통해 웹에 노출할지, JMX를 통해서 노출할지 선택하는것이다. 

(대부분의 엔드포인트는 기본으로 활성화 되어 있으나 노출이 되어 있지 않다.)

### 다양한 엔드포인트 종류

- `beans` : 스프링 컨테이너에 등록된 스프링 빈을 보여준다.
- `conditions` : condition 을 통해서 빈을 등록할 때 평가 조건과 일치하거나 일치하지 않는 이유를 표시한다.
- `configprops` : @ConfigurationProperties 를 보여준다.
- `env` : Environment 정보를 보여준다.
- `health` : 애플리케이션 헬스 정보를 보여준다.
- `httpexchanges` : HTTP 호출 응답 정보를 보여준다. `HttpExchangeRepository` 를 구현한 빈을 별도로 등록해야 한다.
- `info` : 애플리케이션 정보를 보여준다.
- `loggers` : 애플리케이션 로거 설정을 보여주고 변경도 할 수 있다.
- `metrics` : 애플리케이션의 메트릭 정보를 보여준다.
- `mappings` : @RequestMapping 정보를 보여준다.
- `threaddump` : 쓰레드 덤프를 실행해서 보여준다.
- `shutdown` : 애플리케이션을 종료한다. 이 기능은 기본으로 비활성화 되어 있다.

## 헬스 정보

헬스 정보를 사용하면 어플리케이션에 문제가 발생했을 때 문제를 빠르게 인지할 수 있다. db응답이나 디스크 사용량에는 문제가 없는지 같은 정보를 포함해서 만들어진다. 

```yaml
management:
	endpoint:
		health:
			show-details: always
```

- 이렇게 설정을 하면 각각의 항목을 확인할 수 있다.
- 다양한 지원 기능 확인
    
    [Production-ready Features](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.health.auto-configured-health-indicators)
    

## 어플리케이션 정보

`info` 엔드포인트는 애플리케이션의 기본 정보를 노출한다.

- `java` : 자바 런타임 정보
- `os` : OS 정보
- `env` : Environment 에서 info. 로 시작하는 정보
- `build` : 빌드 정보, META-INF/build-info.properties 파일이 필요하다.
- `git` : git 정보, git.properties 파일이 필요하다.
- `env` , `java` , `os` 는 기본으로 비활성화 되어 있다.

## 로거

`logger` 엔드포인트를 사용하면 로깅과 관련된 정보를 확인하고 실시간으로 변경할 수 있다.

## Http 요청 응답 기록

http 요청과 응답 기록을 확인할 수 있다. 

- `HttpExchangeRepository` 인터페이스의 구현체를 빈으로 등록하면httpexchanges 엔드포인트를 사용할 수 있다.
- 이 구현체는 최대 100개의 HTTP 요청을 제공한다. 최대 요청이 넘어가면 과거 요청을 삭제한다.

```java
@SpringBootApplication
public class ActuatorApplication {
	public static void main(String[] args) {
	 SpringApplication.run(ActuatorApplication.class, args);
}

@Bean
public InMemoryHttpExchangeRepository httpExchangeRepository() {
	return new InMemoryHttpExchangeRepository();

	} 
}
```


## 엑츄에이터와 보안

- **보안 주의** : 액츄에이터가 제공하는 기능들은 우리 애플리케이션의 내부 정보를 너무 많이 노출한다. 그래서 외부 인터넷 망이 공개된 곳에 액츄에이터의 엔드포인트를 공개하는 것은 보안상 좋은 방안이 아니다. 따라서  액츄에이터의 엔드포인트들은 외부 인터넷에서 접근이 불가능하게 막고, 내부에서만 접근 가능한 내부망을  사용하는 것이 안전하다.
- **따라서 액츄에이터를 다른 포트에서 실행** : 외부 인터넷 망을 통해서 8080 포트에만 접근할 수 있고, 다른 포트는 내부망에서만 접근 할수 있다면 액츄에이터를 다른 포트로 설정하면 된다.
    - 액츄에이터 포트 설정 : `management.server.port=9292`