# Section 09. 마이크로미터, 프로메테우스, 그라파나


## 마이크로미터 소개

- 서비스를 운영할 때 애플리케이션의 CPU, 메모리, 커넥션 사용, 고객 요청수 같은 수많은 지표들을 확인하는 것이 필요하다.
- 개발자가 개발, 배포, 운영, 모니터링 하는게 좋다.

### 모니터링 툴에 지표 전달
- CPU, JVM, 커넥션 정보 등을 JMS 툴에 전달할 때 각 정보를 JMS 모니터링 툴이 정한 포맷에 맞추어 측정하고 전달해야한다.

### 모니터링 툴 변경
- 그런데 모니터링 툴을 변경하면 코드를 변경해야하는 경우가 생긴다.
- 이러한 문제를 해결하는 것이 마이크로미터(Micrometer)라는 라이브러리다.

### 마이크로미터 추상화

- 마이크로미터는 애플리케이션 메트릭 파사드라고 불린다.
    - 애플리케이션의 메트릭(측정 지표)을 마이크로미터가 정한 표준 방법으로 모아서 제공해준다.
    - 마이크로미터가 추상화를 통해 구현체를 갈아끼울 수 있도록 해두었다.
- 스프링은 마이크로미터를 활용한다. 스프링 액츄에이터는 마이크로미터를 기본으로 내장한다.
    - 로그를 추상화하는 SLF4J와 유사하다.
        - 로그도 구현체는 다양하다.
- 개발자는 마이크로미터가 정한 표준 방식으로 메트릭을 전달한다. 그리고 사용하는 모니터링 툴에 맞는 구현체를 선택한다.

**마이크로미터가 지원하는 모니터링 툴**

- AppOptics
- Atlas
- CloudWatch
- Datadog
- Dynatrace
- Elastic
- Ganglia
- Graphite
- Humio
- Influx
- Instana
- JMX
- KairosDB
- New Relic
- Prometheus
- SignalFx
- Stackdriver
- StatsD
- Wavefront

[](https://micrometer.io/docs)

### 메트릭 확인하기

- 개발자는 각 지표를 직접 수집해서 마이크로미터가 제공하는 표준 방법에 따라 등록하면 된다.
- 마이크로미터는 다양한 지표 수집 기능을 이미 만들어서 제공한다.
- 스프링 부트 액츄에이터는 제공하는 지표 수집을 `@AutoConfiguration`을 통해 자동으로 등록해준다.

**metrics 엔드포인트**

[http://localhost:8080/actuator/metrics](http://localhost:8080/actuator/metrics)

- 액추에이터가 마이크로미터를 통해 등록한 기본 메트릭을 확인할 수 있다.

**자세히 확인하기**

- [http://localhost:8080/actuator/metrics/{name}](http://localhost:8080/actuator/metrics/%7Bname%7D)

### Tag 필터

- availablTags를 보면 다음 항목을 확인할 수 있다.
- tag: area, values[heap, nonheap]
- tag:id, values[G1 Survivor Space, …]

해당 Tag를 기반으로 정보를 필터링해서 확인할 수 있다.

`tag=KEY:VALUE`과 같은 형식을 사용해야 한다.

- [http://localhost:8080/actuator/metrics/jvm.memory.used?tag=area:heap](http://localhost:8080/actuator/metrics/jvm.memory.used?tag=area:heap)
- [http://localhost:8080/actuator/metrics/jvm.memory.used?tag=area:nonheap](http://localhost:8080/actuator/metrics/jvm.memory.used?tag=area:nonheap)


### HTTP 요청수를 확인

- [http://localhost:8080/actuator/metrics/http.server.requests](http://localhost:8080/actuator/metrics/http.server.requests)

HTTP 요청수에서 일부 내용 필터링

- `/log` 요청만 필터
    - [http://localhost:8080/actuator/metrics/http.server.requests?tag=uri:/log](http://localhost:8080/actuator/metrics/http.server.requests?tag=uri:/log)
- `/log` 요청 & HTTP Status  200
    - [http://localhost:8080/actuator/metrics/http.server.requests?tag=uri:/log&tag=status:200](http://localhost:8080/actuator/metrics/http.server.requests?tag=uri:/log&tag=status:200)

### 다양한 메트릭

마이크로미터와 액츄에이터가 기본으로 제공하는 다양한 메트릭을 확인해보자.

- JVM 메트릭
- 시스템 메트릭
- 애플리케이션 시작 메트릭
- 스프링 MVC 메트릭
- 톰캣 메트릭
- 데이터 소스 메트릭
- 로그 메트릭

#### JVM 메트릭

- `jvm.` 으로 시작
- 메모리 및 버퍼풀 세부정보
- 가비지 수집 관련 통계
- 스레드 활용
- 로드 및 언로드된 클래스 수
- JVM 버전 정보
- JIT 컴파일 시간

#### 시스템 메트릭

- `system.`, `process.`, `disk.` 으로 시작
- CPU 지표
- 파일 디스크립터 메트릭
- 가동 시간 메트릭
- 사용 가능한 디스크 공간

#### 애플리케이션 시작 메트릭

- `application.started.time`: 애플리케이션을 시작하는데 걸리는 시간(`ApplicationStartedEvent`로 측정)
- `application.read.time`: 애플리케이션이 요청을 처리할 준비가 되는데 걸리는 시간(`ApplicationReadyEvent` 로 측정)

스프링은 내부에 여러 초기화 단계가 있고 각 단계별로 내부에서 애플리케이션 이벤트를 발행한다.

- `ApplicationStartedEvent`: 스프링 컨테이너가 완전히 실행된 상태이다. 이후에 커맨드 라인 러너가 호출된다.
- `ApplicationReadyEvent`: 커맨드 라인 러너가 실행된 이후에 호출된다.

#### 스프링 MVC 메트릭

스프링 MVC 컨트롤러가 처리하는 모든 요청

메트릭 이름: `http.server.requests`

TAG를 사용해서 정보 분류해서 확인 가능

- uri: 요청 URI
- method: GET, POST 같은 HTTP 메서드
- status: 200, 400, 500 같은 HTTP status 코드
- exception: 예외
- outcome: 상태코드를 그룹으로 모아서 확인 `1xx:INFORMATIONAL`, `2xx:SUCCESS`, `3xx:REDIRECTION`, `4xx:CLIENT_ERROR`, `5xx:SERVER_ERROR`

#### 데이터소스 메트릭

datasource, 커넥션 풀에 관한 메트릭을 확인할 수 있다.

`jdbc.connections.`으로 시작

- 최대 커넥션, 최소 커넥션, 활성 커넥션, 대기 커넥션 수 등을 확인 가능
- 히카리 커넥션 풀을 사용하면 `hikaricp.`를 통해 자세한 메트릭을 확인할 수 있다.

#### 로그 메트릭

- `logback.event`: loback 로그에 대한 메트릭을 확인할 수 있다.
- `trace`, `debug`, `info`, `warn`, `error` 각각의 로그 레벨에 따른 로그 수를 확인할 수 있다.
    - `error` 로그가 급격히 높아지면 위험한 신호로 확인할 수 있다.

#### 톰캣 메트릭

- 톰캣 메트릭은 tomcat. 으로 시작한다.
- 다음 yml 옵션을 켜야한다.(tomcat.session. 관련 정보만 노출)
    
    applcation.yml
    
    ```json
    server:
    	tomcat:
    		mbeanregistry:
    			enabled: true
    ```
    
    - 톰캣의 최대 쓰레드, 사용 쓰레드 수를 포함한 다양한 메트릭을 확인할 수 있다.

#### 기타

- HTTP 클라이언트 메트릭(RestTemplate, WebClient)
- 캐시 메트릭
- 작업 실행과 스케줄 메트릭
- 스프링 데이터 리포지토리 메트릭
- 몽고 DB 메트릭
- 레디스 메트릭

#### 사용자 정의 메트릭

- 사용자가 직접 메트릭을 정의할 수 있다.
    - 주문수, 취소수

#### 정리

- 액츄에이터를 통해 수많은 메트릭이 자동으로 만들어져야한다.
- 지속해서 보관해야 과거의 데이터들도 확인할 수 있다.
    - 메트릭을 지속적으로 수집하고 보관할 데이터베이스가 필요하다.
    - 메트릭을 그래프를 통해 한눈에 쉽게 확인할 수 있는 대시보드가 필요하다.

참고: 지원하는 다양한 메트릭들은 다음 공식 메뉴얼을 참고하자

> [https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.supported](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.supported)

## 프로메테우스와 그라파나 소개

### 프로메테우스

- 애플리케이션에서 발생한 메트릭을 그 순간만 확인하는 것이 아니라 과거 이력까지 함께 확인하려면 메트릭을 보관하는 DB가 필요하다.
- 메트릭을 지속해서 수집하고 DB에 저장해야한다. 이 역할을 프로메테우스가 한다.

### 그라파나

- DB에 있는 데이터를 불러서 사용자가 보기 편하게 보여주는 대시보드가 필요하다.
- 그라파타는 유연하며 데이터를 그래프로 보여주는 툴이다.
- 수많은 그래프를 제공하고 프로메테우스를 포함한 다양한 데이터소스를 지원한다.


전체구조
1. 스프링 부트 액츄에이터와 마이크로미터를 사용하면 수많은 메트릭을 자동으로 생성한다.
    1. 마이크로미터 프로메테우스 구현체는 프로메테우스가 읽을 수 있는 포맷으로 메트릭을 생성한다.
2. 프로메테우스는 이렇게 만들어진 메트릭을 지속해서 수집한다.
3. 프로메테우스는 수집한 메트릭을 내부 DB에 저장한다.
4. 사용자는 그라파나 대시보드 툴을 통해 그래프로 편리하게 메트릭을 조회한다. 이때 필요한 데이터는 프로메테우스를 통해서 조회한다.

## 프로메테우스 - 기본 기능

- 태그, 레이블: error, exception, instance, job, method, outcome, status, uri는 각각의 메트릭 정보를 구분해서 사용하기 위한 태그이다. 마이크로미터에서는 이것을 태그(tag)라고 하고 프로메테우스 에서는 이를 레이블(Label)이라고 한다.
- 숫자: 끝에 마지막에 254와 같은 숫자가 메트릭 값이다.
- Table → evalutation time 을 수정해서 과거 시간 조회 가능
- Graph → 메트릭을 그래프로 조회 가능

## 필터

레이블을 기준으로 필터 사용. 필터는 중괄호(`{}`) 문법을 사용

레이블 일치 연산자

- `=` 제공된 문자열과 정확히 동일한 레이블 선택
- `!=` 제공된 문자열과 같지 않은 레이블
- `=~` 제공된 문자열과 정규식 일치하는 레이블 선택
- `!~` 제공된 문자열과 정규식 일치하지 않는 레이블 선택

예)

- `uri=/log`,`method=GET` 조건으로 필터
    - http_server_requests_seconds_count{uri="/log", method="GET"}
- `/actuator/prometheus`는 제외한 조건으로 필터
    - `http_server_requests_seconds_count{uri!="/actuator/prometheus"}`
- method가 `GET`,`POST`인 경우를 포함해서 필터
    - `http_server_requests_seconds_count{method=~"GET|POST"}`
- `/actuator`로 시작하는 `uri`는 제외한 조건으로 필터
    - `http_server_requests_seconds_count{uri!~"/actuator.*"}`

### 연산자 쿼리와 함수

다음과 같은 연산자를 지원한다.

- +
- -
- *
- /
- %
- ^

- sum
    - 값의 합계를 구한다.
        - 예) sum(http_server_requests_seconds_count)
- sum by
    - SQL의 group by 기능과 유사하다.
        - 예)sum by(method, status)(http_server_requests_seconds_count)
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6d284f75-88b3-4103-b576-15bebcfb8335/Untitled.png)
        
- count
    - count(http_server_requests_seconds_count)
    - 매트릭 자체의 수 카운트
- topk
    - topk(3, http_server_requests_seconds_count)
    - 상위 3개 메트릭 조회
- 오프셋 수정자
    - http_server_requests_seconds_count offset 10m
    - offset 10m과 같이 나타낸다. 현재를 기준으로 특정 과거 시점의 데이터를 반환
- 범위 벡터 선택기
    - http_server_requests_seconds_count[1m]
        - 마지막에 [1m], [60s]와 같이 표현. 지난 1분간의 모든 기록값을 선택
        - 범위 벡터 선택기는 차트에 바로 표현할 수 없다. 데이터로만 확인이 가능하다.
        - 범위 벡터 선택의 결과를 차트로 보기 위해서는 가공이 필요하다. 상대적인 증가 확인 방법을 참고해야한다.

## 프로메테우스 - 게이지와 카운터

메트릭은 크게 보면 게이지와 카운터라는 2가지로 분류할 수 있다.

### 게이지(Gauge)

- 임의로 오르내릴 수 있는 값
    - CPU 사용량, 메모리 사용량, 사용중인 커넥션
- CPU 사용량의 현재 상태를 계속 측정하고 그 값을 그대로 그래프에 출력하면 과거부터 지금까지의 CPU 사용량을 확인할 수 있다.

### 카운터(Counter)

- 단순하게 증가하는 단일 누적값
    - HTTP 요청 수, 로그 발생 수

- 계속 누적해서 증가하는 값이다.
- 하지만 이렇게 증가만하는 그래프에서 특정 시간에 얼마나 고객 요청이 들어왔는지 한눈에 확인하기 어렵다. 이런 문제를 해결하기 위해 `increase()`, `rate()` 함수를 지원한다.

### increase()

- 시간 단위별로 증가를 확인할 수 있다.
    - `increase(http_server_requests_seconds_count{uri="/log"}[1m])`

- 분당 얼마나 고객의 요청이 증가했는지 한 눈에 확인할 수 있다.

### rate()

범위 백터에서 초당 평균 증가율을 계산한다.

- increase()는 숫자를 직접 카운트하고 rate()는 초당 평균을 나누어서 계산한다.
- rate(data[1m] 에서 [1m]이라고 하면 60초가 기준이 되므로 60을 나눈 수이다.
- rate(data[2m] 에서 [2m]이라고 하면 120초가 기준이 되므로 120을 나눈  수이다.

### irate()

- rate와 유사한데 범위 백터에서 초당 순간 증가율을 계산한다.
- 급격하게 증가한 내용을 확인하기 좋다.

- 기본기능: [https://prometheus.io/docs/prometheus/latest/querying/basics/](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- 연산자: [https://prometheus.io/docs/prometheus/latest/querying/operators/](https://prometheus.io/docs/prometheus/latest/querying/operators/)
- 함수: [https://prometheus.io/docs/prometheus/latest/querying/functions/](https://prometheus.io/docs/prometheus/latest/querying/functions/)

프로메테우스의 단점은 한눈에 들어오는 대시보드를 만들어보기 어렵다는 점이다. 이 부분은 그라파나를 사용하면 된다.
