# Section 8. Actuator

[Production-ready Features](https://docs.spring.io/spring-boot/docs/3.0.1/reference/html/actuator.html)

## 프러덕션 준비 기능

운영 환경에서 서비스에 문제가 없는 지 모니터링하고 지표를 심어서 감시하는 활동 및 기능들을 프로덕션 준비 기능이라 한다.

- 지표(metric), 추적(trace), 감사(auditing)
- 모니터링

스프링 부트가 제공하는 엑추에이터는 이런 기능들을 매우 편리하게 사용할 수 있는 편리한 기능을 제공한다.

## 액츄에이터 시작

### 동작확인

`http://localhost:8080/actuator`로 get 요청을 보내본다.

```json
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

web 모든 actuactor의 기능을 보기 위한 설정

```yaml
management:
  endpoint:
    web:
      exposure:
        include: "*"
```

- `http://localhost:8080/actuator`

    ```json
    {
      "_links": {
        "self": {
          "href": "http://localhost:8080/actuator",
          "templated": false
        },
        "beans": {
          "href": "http://localhost:8080/actuator/beans",
          "templated": false
        },
        "caches-cache": {
          "href": "http://localhost:8080/actuator/caches/{cache}",
          "templated": true
        },
        "caches": {
          "href": "http://localhost:8080/actuator/caches",
          "templated": false
        },
        "health-path": {
          "href": "http://localhost:8080/actuator/health/{*path}",
          "templated": true
        },
        "health": {
          "href": "http://localhost:8080/actuator/health",
          "templated": false
        },
        "info": {
          "href": "http://localhost:8080/actuator/info",
          "templated": false
        },
        "conditions": {
          "href": "http://localhost:8080/actuator/conditions",
          "templated": false
        },
        "configprops-prefix": {
          "href": "http://localhost:8080/actuator/configprops/{prefix}",
          "templated": true
        },
        "configprops": {
          "href": "http://localhost:8080/actuator/configprops",
          "templated": false
        },
        "env": {
          "href": "http://localhost:8080/actuator/env",
          "templated": false
        },
        "env-toMatch": {
          "href": "http://localhost:8080/actuator/env/{toMatch}",
          "templated": true
        },
        "loggers": {
          "href": "http://localhost:8080/actuator/loggers",
          "templated": false
        },
        "loggers-name": {
          "href": "http://localhost:8080/actuator/loggers/{name}",
          "templated": true
        },
        "heapdump": {
          "href": "http://localhost:8080/actuator/heapdump",
          "templated": false
        },
        "threaddump": {
          "href": "http://localhost:8080/actuator/threaddump",
          "templated": false
        },
        "metrics-requiredMetricName": {
          "href": "http://localhost:8080/actuator/metrics/{requiredMetricName}",
          "templated": true
        },
        "metrics": {
          "href": "http://localhost:8080/actuator/metrics",
          "templated": false
        },
        "scheduledtasks": {
          "href": "http://localhost:8080/actuator/scheduledtasks",
          "templated": false
        },
        "mappings": {
          "href": "http://localhost:8080/actuator/mappings",
          "templated": false
        }
      }
    }
    ```


### 엔드포인트 설정

엔트포인트를 사용하기 위한 과정은 2가지

1. 엔드포인트 활성화
2. 엔드포인트 노출

대부분 기능은 활성화 되어 있다.

노출은 HTTP, JMX

## 다양한 엔드포인트

주로 사용하는 엔드포인트들

- beans: 스프링 컨테이너에 등록된 스프링 빈을 보여준다.
- conditions: `condition에 대한 매칭 정보를 보여준다.`
- configprops: `@ConfigurationProperties` 를 보여준다.
- env: `Environment` 정보를 보여준다.
- health: 어플리케이션 헬스 정보를 보여준다.
- httpexchanges: HTTP 호출 응답 정보를 보여준다. `HttpExchangeRepository`를 구현한 빈을 별도로 등록해야 한다.
- info: 애플리케이션 정보를 보여준다.
- loggers: 애플리케이션 로고 설정에 대해서 보여준다.
- matrics: 다양한 매트릭트 정보를 볼 수 있다.
- mappings: `@RequstMapping` 정보를 보여준다.
- threaddump: 쓰레드 덤프를 실행해서 보여준다.

## 헬스 정보

애플리케이션에 문제가 발생하면 빠르게 인지할 수 있다.

`http://localhost:8080/actuator/health`

```json
{
	"status": "UP"
}
```

### 더 다양한 값을 보기 위한 설정

```yaml
management:
	endpoint:
	    health:
	      show-details: always
```

```json
{
	"status": "UP",
	"components": {
		"db": {
			"status": "UP",
			"details": {
				"database": "H2",
				"validationQuery": "isValid()"
			}
		},
		"diskSpace": {
			"status": "UP",
			"details": {
				"total": 494384795648,
				"free": 328167194624,
				"threshold": 10485760,
				"path": "/Users/dojinyou/Projects/dojinyou/SpringBoot-JPA-RoadMap/springboot-key-principle-and-application/w6/.",
				"exists": true
			}
		},
		"ping": {
			"status": "UP"
		}
	}
}
```

### component만 보기

```yaml
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

### 헬스 이상 상태

컴포넌트 중 하나만 `DOWN`되면 전체 상태가 `DOWN`이 된다.

### 애플리케이션 정보

`http://localhost:8080/actuator/info`

기본제공 기능

- java: 자바 런타임 정보
- os: OS 정보
- env: `Environment`에서 `info.` 로 시작하는 정보
- build: 빌드 정보, `META-INF/build-info.properties` 확인이 필요하다.
- git: git 정보, `git.properties` 파일이 피료

- env, java, os는 기본으로 비활성화되어 있다.

Java, OS 정보 확인하기

- 설정파일

    ```yaml
    management:
      info:
        java:
          enabled: true
        os:
          enabled: true
    ```

- 응답결과

    ```yaml
    {
      "java": {
        "version": "17.0.5",
        "vendor": {
          "name": "Amazon.com Inc.",
          "version": "Corretto-17.0.5.8.1"
        },
        "runtime": {
          "name": "OpenJDK Runtime Environment",
          "version": "17.0.5+8-LTS"
        },
        "jvm": {
          "name": "OpenJDK 64-Bit Server VM",
          "vendor": "Amazon.com Inc.",
          "version": "17.0.5+8-LTS"
        }
      },
      "os": {
        "name": "Mac OS X",
        "version": "13.3.1",
        "arch": "aarch64"
      }
    }
    ```


Env 설정 확인하기

- 설정파일

    ```yaml
    management:
      info:
        env:
          enabled: true
    
    info: # info 아래의 값들이 출력된다.
      app:
        name: hello-actuator
    ```

- 응답결과

    ```yaml
    {
      "app": {
        "name": "hello-actuator"
      }
    }
    ```


Build 정보 확인하기

- 설정하기
    - build.gradle 빌드 정보 추가하기

        ```kotlin
        springBoot {
            buildInfo()
        }
        ```

    - 설정 파일

        ```yaml
        management:
          info:
            build:
              enabled: true
        ```

- 응답결과

    ```json
    {
      "build": {
        "artifact": "w6",
        "name": "w6",
        "time": "2023-04-23T01:42:55.847Z",
        "version": "0.0.1-SNAPSHOT",
        "group": "com.dojinyou.inflearn.roadmap"
      }
    }
    ```


git 정보 확인하기

- 설정하기
    - 깃으로 관리하기
    - 플러그인 추가

        ```kotlin
        plugins {
            id("com.gorylenko.gradle-git-properties") version "2.4.1"
        }
        ```

    - 설정파일 추가

        ```yaml
        management:
          info:
            git:
              enabled: true
        ```

- 응답결과

    ```json
    {
      "git": {
        "branch": "springboot-key-principle-and-application/5w",
        "commit": {
          "id": "c242a2c",
          "time": "2023-04-22T16:42:26Z"
        }
      },
      "build": {
        "artifact": "w6",
        "name": "w6",
        "time": "2023-04-23T01:55:14.939Z",
        "version": "0.0.1-SNAPSHOT",
        "group": "com.dojinyou.inflearn.roadmap"
      }
    }
    ```


## Logger

- logging setting

    ```kotlin
    @RestController
    class LoggerController {
        private val logger = LoggerFactory.getLogger(this::class.java)
    
        @RequestMapping("/log")
        fun log(): String {
            logger.trace("trace log")
            logger.debug("debug log")
            logger.info("info log")
            logger.warn("warn log")
            logger.error("error log")
            return "OK"
        }
    }
    ```

- 로깅레벨 변경하기

    ```yaml
    logging:
      level:
        com.dojinyou.inflearn.roadmap.w6: debug
    ```

  설정 파일을 통해 특정 패키지의 로그 레벨 설정 가능

- loggers actuator 호출

  `http://localhost:8080/actuator/loggers`

    ```yaml
    {
      "levels": [
        "OFF",
        "ERROR",
        "WARN",
        "INFO",
        "DEBUG",
        "TRACE"
      ],
      "loggers": {
    		"ROOT": {
    		  "configuredLevel": "INFO",
    		  "effectiveLevel": "INFO"
    		},
    		...,
    		"com": {
          "effectiveLevel": "INFO"
        },
        "com.dojinyou": {
          "effectiveLevel": "INFO"
        },
        "com.dojinyou.inflearn": {
          "effectiveLevel": "INFO"
        },
        "com.dojinyou.inflearn.roadmap": {
          "effectiveLevel": "INFO"
        },
        "com.dojinyou.inflearn.roadmap.w6": {
          "configuredLevel": "DEBUG",
          "effectiveLevel": "DEBUG"
        },
        "com.dojinyou.inflearn.roadmap.w6.LoggerController": {
          "effectiveLevel": "DEBUG"
        },
        "com.dojinyou.inflearn.roadmap.w6.W6ApplicationKt": {
          "effectiveLevel": "DEBUG"
        },
    		...,
    	}
    	"groups": {}
    }
    ```

  기본 `ROOT` 패키지 설정은 `INFO`이고 이하 패키지들은 이 영향으로 `INFO`로 로그 레벨 설절

  설정한 패키지인 `com.dojinyou.inflearn.roadmap.w6` 에는 `configuredLevel`이 존재

  이하 레벨은 영향을 받아 `effectiveLevel`이 `DEBUF`로 되어 있다.

  ### 실시간 로그 레벨 변경

  일반적으로 로깅 설정을 변경하기 위해서는 설정값을 바꾸고 `서버를 다시 시작`해야 한다.

  하지만, loggers 같은 endpoint를 이용하면 post 요청으로 메모리 상의 값을 변경할 수 있다.

    ```yaml
    POST http://localhost:8080/actuator/loggers/{packacge path} HTTP/1.1
    Content-Type: application/json
    
    {
    	"configuredLevel": "TRACE"
    }
    ```

  ## HTTP 요청 응답 기록

  `httpexchanges` 엔드포인트를 이용해 http의 요청과 응답의 과거 기록을 확인할 수 있다.

  `HttpExchangeRepository` 인터페이스의 구현체를 `Bean`으로 등록하면 `httpexchanges` 사용

  (해당 빈이 등록되지 않으면 엔드포인트가 활성화되지 않는다.)

  스프링 부트는 기본으로 `InMemoryHttpExchangeRepository` 구현체를 제공한다.

  해당 기능은 매우 단순하고 제한이 많기 때문에, 실제 운영 서비스에서는 모니터링 툴이나 핀포인트, Zipkin 같은 다른 기술을 사용하는 것이 좋다.

  ## 액츄에이터와 보안

  액츄에이터가 제공하는 기능들은 우리 애플리케이션의 내부 정보를 너무 많이 노출하므로 private netwtork에서만 접속할 수 있도록 해야 한다.

  8080포트를 외부에서 접근하도록 열고 다른 포트에 액츄에이터를 설정하면 된다.

  포트 설정

    ```yaml
    management:
      server:
        port: 8081
    ```

  포트 분리가 어렵고 외부에서 접근해야하는 불가피한 상황이 있다면 필터, 인터셉터, 스프링 시큐리티 등을 이용해 인증된 사용자만 접근가능하도록 추가 개발이 필요하다.
