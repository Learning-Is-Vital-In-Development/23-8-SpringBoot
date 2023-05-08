## Overview

전투에서 실패한 지휘관을 용서할 수 있지만 경계에서 실패하는 지휘관은 용서할 수 없다. 서비스를 운영하는 개발자에게 장애는 언제든지 발생할 수 있지만 모니터링 서비스를 구축해서 잘 대응하는 것이 중요하다.

개발자는 애플리케이션을 개발할 때 기능 요구사항만 개발하는 것이 아니다. 서비스에 문제가 없는지 모니터링하고 지표들을 심어 감시하는 활동이 필요하다.

운영환경에서 서비스할 때 필요하는 기능들을 프로덕션 준비 기능이라고 한다. 프로덕션을 운영에 배포할 때 준비해야하는 비 기능적 요소를 뜻한다.

- 지표(metric), 추적(trace), 감사(auditing)
- 모니터링

애플리케이션이 살아있는지, 로그 정보는  정상 설정되어 있는지, 커넥션 풀은 얼마나 사용되고 있는지 확인을 해야할 것이다.

액츄에이터는 이런 기능을 사용할 수 있도록 다양하게 편의를 제공한다. 더 나아가 마이크로미터, 프로메테우스, 그라파나 같은 모니터링 시스템과 쉽게 연동할 수 있는 기능을 제공한다. 액츄에이터는 시스템을 움직이거나 제어하는데 쓰이는 기계장치라는 뜻이다.

# 액츄에이터 시작

---

```java
implementation 'org.springframework.boot:spring-boot-starter-actuator' //actuator 추가
```

[`http://localhost:8080/actuator`](http://localhost:8080/actuator)

```java
{
    "_links": {
        "self": {
            "href": "http://localhost:8080/actuator",
            "templated": false
        },
        "health-path": {
            "href": "http://localhost:8080/actuator/health/{*path}",
            "templated": true
        },
        "health": {
            "href": "http://localhost:8080/actuator/health",
            "templated": false
        }
    }
}
```

- 액튜에이터는 /actuator 경로를 통해 기능을 제공한다.

[`http://localhost:8080/actuator/health`](http://localhost:8080/actuator/health)

health는 현재 서버가 잘 작동하고 있는 애플리케이션의 헬스 상태를 나타낸다.
이외에도 다른 정보들을 확인할 수 있다.

```java
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

- 액츄에이터가 제공하는 수많은 기능을 확인할 수 있다. 이러한 기능 하나하나를 엔드포인트라고 한다.
- 각 엔드포인트는 /actuator/{엔드포인트명} 과 같은 형식으로 접근할 수 있다.

## 엔드포인트 설정

엔드포인트를 사용하기 위해 2가지 설정이 필요하다.

1. 엔드포인트 활성화
    - 기능을 사용할지 on, off를 선택하는 것이다.
2. 엔드포인트 노출
    - 엔드포인트를 HTTP에 노출할지 JMX에 선택할지 선택
    - HTTP로 노출할지 JMX를 통해서 노출할지 둘다 노출할지 위치 지정
    - 활성화가 되어있지 않으면 노출이 안된다.

엔드포인트는 기본으로 활성화되어있다.

- shutdown 기본으로 제외되어있다. 설정을 하기 위해서는 아래와 같이 설정한다.
    
    ```json
    management:
      endpoint:
        shutdown:
          enabled: true
    ```
    
    - [http://localhost:8080/actuator/shutdown](http://localhost:8080/actuator/shutdown) `POST` 호출하면 외부에서 서비스를 종료시킬 수 있다.
    - 이 기능은 주의해서 사용해야한다. 그래서 기본으로 비활성화 되어있다.
- 따라서 노출할지만 선택하면 된다.

```json
management:
  endpoints:
    jmx:
      exposure:
        include: "health,info"
    web:
      exposure:
        include: "*"
        exclude: "env,beans"
```

## 다양한 엔드포인트

엔드포인트 목록

- beans: 스프링 컨테이너에 등록된 스프링 빈을 보여준다.
- conditions: condition을 통해서 빈을 등록할 때 평가조건과 일치하거나 일치하지 않는 이유를 표시한다.
- configprops: @ConfigurationProperties 보여준다.
- env: Environment 정보를 보여준다.
- health: 애플리케이션 헬스 정보를 보여준다.
- httpexchanges: HTTP 호출 응답 정보를 보여준다. `HttpExchangeRepository`를 구현한 빈을 별도로 등록해야한다.
- info: 애플리케이션 정보를 보여준다.
- loggers: 애플리케이션 로거 설정을 보여주고 변경도할 수 있다.
- metrics: 애플리케이션의 메트릭 정보를 보여준다.
- mappings: @RequestMapping 정보를 보여준다.
- threaddump: 스레드 덤프를 실행해서 보여준다.
- shotdown: 애플리케이션을 종료한다. 이 기능은 기본으로 비활성화 되어 있다.

[Production-ready Features](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints)

## 헬스 정보

```json
{
    "status": "UP"
}
```

[http://localhost:8080/actuator/health](http://localhost:8080/actuator/health) GET

헬스 정보를 자세히 보려면 다음 정보를 지정한다.

```json
management:
  endpoint:
    health:
      show-details: always
```
- 각 db마다 health를 확인해주는 validationQuery 가 구현되어있다. 예전에는 없었다.

```json
management:
  endpoint:
    health:
      show-components: always
```

```json
{
    "status": "UP",
    "components": {
        "db": {
            "status": "UP"
        },
        "diskSpace": {
            "status": "UP"
        },
        "ping": {
            "status": "UP"
        }
    }
}
```

- 각 헬스 컴포넌트만 간략하게 노출하고 하나라도 문제가 있으면 전체 상태는 DOWN이 된다.
- 액츄에이터는 db, mong, redis, diskpace, ping 과 같이 수많은 기능을 기본으로 제공한다.

[Production-ready Features](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.health.auto-configured-health-indicators)

헬스 기능은 직접 구현해서 추가할 수 있다.

[Production-ready Features](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.health.writing-custom-health-indicators)

## 애플리케이션 정보

info 엔드포인트는 애플리케이션의 기본 정보를 노출한다.

기본으로 제공하는 기능은 다음과 같다.

- java: 자바 런타임 정보
- os: OS 정보
- env: Environment에서 info.로 시작하는 정보
- build: 빌드 정보, META-INF/build-info.properties 파일이 필요하다.
- git: git 정보, [git.properties](http://git.properties) 파일이 필요하다.

env, java, os는 기본으로 비활성화 되어있다.
```json
management:
  info:
    java:
      enabled: true
		os:
      enabled: true
		env:
      enabled: true

info:
  app:
    name: hello-actuator
    company: ep
```

build 정보

`build.gradle`

```json
springBoot {
    buildInfo()
}
```

git 정보

```json
id "com.gorylenko.gradle-git-properties" version "2.4.1" //git info
```

## 로거

loggers 엔드포인트를 사용해서 로깅과 관련된 정보를 확인하고 설정을 바꿀 수 있다.

application.yml

```java
logging:
  level:
    hello.controller: debug
```

패키지 경로에 따라서 노출되는 로그레벨을 지정할 수 있다.

[`http://localhost:8080/actuator/loggers`](http://localhost:8080/actuator/loggers)
- 로그를 별도로 설정하지 않으면 기본으로 INFO를 사용한다. ROOT의 configuredLevel이 INFO로 설정되어있다.

더 자세히 조회하기

[http://localhost:8080/actuator/loggers/hello.controller](http://localhost:8080/actuator/loggers/hello.controller)

```json
{
    "configuredLevel": "DEBUG",
    "effectiveLevel": "DEBUG"
}
```

### 실시간 로그 레벨 변경

- 개발서버는 보통 DEBUG 로그를 사용하지만 운영 서버는 보통 요청이 아주 많다. 따라서 로그도 너무 많이 남기 때문에 DEBUG 로그까지 모두 출력하게 되면 성능이나 디스크에 영향을 주게 된다. 운영 서버는 중요하다고 판단되는 INFO 로그레벨을 사용한다.
- 서비스 운영중에 금하게 로그레벨을 바꿔야할 때, 액츄에이터로 변경할 수 있다.

POST [http://localhost:8080/actuator/loggers/hello.controller](http://localhost:8080/actuator/loggers/hello.controller)

raw json body

```java
{
	"configuredLevel": "TRACE"
}
```

## HTTP 요청 응답 기록

- http 요청과 응답의 과거 기록을 확인하고 싶다면 httpexchanges 엔드포인트를 사용하면 된다.
- HttpExchangeRepository 인터페이스의 구현체를 빈으로 등록하면 httpexchanges 앤드포인트를 사용할 수 있다.
- 빈을 등록하지 않으면 httpexchanges 엔드포인트가 활성화되지 않는다.
- 스프링 부트는 InMemoryHttpExchangeRepository 구현체를 제공한다.

[`http://localhost:8080/actuator/httpexchanges`](http://localhost:8080/actuator/httpexchanges)

```java
{
    "exchanges": [
        {
            "timestamp": "2023-05-06T02:40:01.035050Z",
            "request": {
                "uri": "http://localhost:8080/actuator/httpexchange4s",
                "method": "GET",
                "headers": {
                    "content-type": [
                        "application/json"
                    ],
                    "user-agent": [
                        "PostmanRuntime/7.32.2"
                    ],
                    "accept": [
                        "*/*"
                    ],
                    "cache-control": [
                        "no-cache"
                    ],
                    "postman-token": [
                        "e5dc2b1c-05f5-4cc7-b290-51ea4c4e5312"
                    ],
                    "host": [
                        "localhost:8080"
                    ],
                    "accept-encoding": [
                        "gzip, deflate, br"
                    ],
                    "connection": [
                        "keep-alive"
                    ],
                    "content-length": [
                        "30"
                    ]
                }
            },
            "response": {
                "status": 404,
                "headers": {
                    "Vary": [
                        "Origin",
                        "Access-Control-Request-Method",
                        "Access-Control-Request-Headers"
                    ]
                }
            },
            "timeTaken": "PT0.033546S"
        }
    ]
}
```
- 실제 운영 서비스에서는 모니터링 투리, 핀포인트, Zipkin 등의 기술을 사용하는 것이 좋다.

## 액츄에이터와 보안

보안주의

- 액츄에이터의 엔드포인트를 공개하는 것은 보안상 좋은 방안이 아니다.
- 액츄에이터의 엔드포인트들은 외부 인터넷에서 접근이 불가능하게 막고 내부에서만 접근 가능한 내부망을 사용하는 것이 안전하다.

액츄에이터를 다른 포트에서 실행

- 액츄에이터는 다른 포트를 설정하면 된다.
- 이경우 8080 포트에서 액츄에이터를 접근할 수 없다.

```java
management.server.port=9292
```

액츄에이터 URL 경로에 인증 설정

- 포트 분리가 어렵고 외부 인터넷 망을 통해서 접근해야 한다면 /actuator 경로에 서블릿 필터, 스프링 인터셉터 또는 스프링 시큐리티를 통해 인증된 사용자만 접근 가능하도록 추가 개발이 필요하다.

앤드포인트 경로 변경

```java
management:
	endpoints:
		web:
			base-path: "/manage"
```

