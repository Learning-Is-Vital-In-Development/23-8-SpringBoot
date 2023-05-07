# Section8: 액츄에이터

실제 서비스의 운영환경에서는 비 기능적 요소인 프로덕션 준비 기능을 필수적으로 준비해야 한다.

- 지표(metric), 추적(trace), 감사(auditing)
- 모니터링

애플리케이션이 현재 살아있는지, 로그 정보는 정상 설정 되었는지, 커넥션 풀은 얼마나 사용되고 있는지 등을 확인할 수 있어야 한다.

**build.gradle - 의존성 추가**

```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator' //**actuator 추가**
```

액츄에이터는 /actuator 경로를 통해서 기능을 제공

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

### **액츄에이터 기능을 웹에 노출**

**application.yml - 추가**

```yaml
management:
    endpoints:
      web:
        exposure:
		  include: "*"
```

- 액츄에이터가 제공하는 기능 하나하나를 엔드포인트라고 한다.
- 각각의 엔트포인트는 `/actuator/{엔드포인트명}` 과 같은 형식으로 접근할 수 있다.

### 엔트포인트 설정

엔트포인트 사용을 위해 2가지 과정을 진행한다.

1. 엔트포인트 활성화
2. 엔트포인트 노출

**대부분의 엔드포인트는 기본으로 활성**화 되어 있기 때문에 어떤 것을 노출할지 선택하면 된다.

특정 엔드포인트를 활성화 하려면 `management.endpoint.{엔드포인트명}.enabled=true` 를 적용하면 된다.

## 헬스 정보

헬스 정보를 사용하면 애플리케이션에 문제가 발생했을 때 문제를 빠르게 인지할 수 있다.

헬스 정보는 애플리케이션이 사용하는 데이터베이스가 응답하는지, 디스크 사용량에는 문제가 없는지 같은 다양한 정보들을 포함해서 만들어진다.

**자세한 헬스 정보 보기**

```yaml
management:
    endpoint:
      health:
        show-details: always
```

- `show-details` 말고 `show-componets`로 할 시 각 헬스 컴포넌트의 상태 정보만 간략하게 노출

**헬스 이상 상태**

헬스 컴포넌트 중 하나라도 문제가 있을 시 전체 상태가 `DOWN`이다.

### 애플리케이션 정보

`info` 엔드포인트는 애플리케이션의 기본 정보를 노출한다.

**기본 제공 기능**

- `java` : 자바 런타임 정보
- `os`: OS 정보
- `env` : `Environment` 에서 `info.` 로 시작하는 정보
- `build` : 빌드 정보, `META-INF/build-info.properties` 파일이 필요하다.
- `git` : `git` 정보, `git.properties` 파일이 필요하다.

`env`, `java`, `os`는 기본으로 비활성화 되어 있다.

→ `management.info.<id>.enabled` 의 값을 true 로 지정하면 활성화

빌드 정보를 노출하려면 빌드 시점에 META-INF/build-info.properties 파일을 만들어야 한다.

**build.gradle - 빌드 정보 추가**

```groovy
springBoot {
      buildInfo()
}
```

git 정보를 노출하려면 git.properties 파일이 필요하다.

**build.gradle - git 정보 추가**

```groovy
plugins {
		id "com.gorylenko.gradle-git-properties" version "2.4.1" //git info
}
```

### 로거

loggers 엔드포인트를 사용하면 로깅과 관련된 정보를 확인하고, 또 실시간으로 변경할 수도 있다.

로그를 별도로 설정하지 않으면 스프링 부트는 기본으로 INFO 를 사용한다.

**실시간 로그 레벨 변경**

보통 운영서버는 중요하다고 판단되는 `INFO` 로그 레벨을 사용한다.

서비스 운영 중 `DEBUG` 나 `TRACE` 로그를 남겨서 확인해야 확인하고 싶을 때 `loggers` 엔드포인트를 사용하면 애플리케이션을 다시 시작하지 않고, 실시간으로 로그 레벨을 변경할 수 있다.

## 액츄에이터와 보안

**보안 주의**

액츄에이터의 엔드포인트들은 외부 인터넷에서 접근이 불가능하게 막고, 내부에서만 접근 가능한 내부망을 사용하는 것이 안전하다.
