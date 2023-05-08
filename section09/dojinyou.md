# Section 9. 마이크로미터, 프로메테우스, 그라파나

# 마이크로미터([Micrometer](https://micrometer.io/))

모니터링 툴이 작동하려면 시스템의 다양한 지표들을 각각의 모니터링 툴에 맞도록 만들어서 보내주어야 한다.

예를 들어 CPU, JVM, 커넥션 정보 등을 JMX 모니터링 툴에 전달한다고 가정해보자. 그러면 각각의 정보를 JMX 모니터링 툴이 정한 포맷에 맞게 측정하고 전달해야 한다. 만약 모니터링 툴을 바꾼다고 기존에 측정하던 방식을 새로운 모니터링 툴에 맞추어 변경해야 한다.

이런 문제를 해결하는 것인 바로 마이크로미터(Micrometer)라는 라이브러리이다.

### 마이크로미터 추상화

- 마이크로 미터의 표준 추상화 방식에 따라 우리는 값을 넣어주기만 하면, 각각의 툴에 맞는 구현체가 구현되어 있어서 해당 구현체가 특정 툴에 맞도록 매트릭을 변환하여 연결을 도와 준다.
- 스프링부트 액츄에이터는 마이크로미터를 기본으로 내장해서 사용한다.
- 개발자는 마이크로미터가 정한 표준 방법으로 메트릭을 전달하면 된다. 그리고 사용하는 모니터링 툴에 맞는 구현체를 선택하면 된다.
- 마이크로미터가 지원하는 모니터링 툴(중 유명한거)
    - Atlas
    - CloudWatch
    - Datadog
    - Elastic
    - Prometheus

## 매트릭 확인하기

다양한 지표들을 수집하는 것조차 어려운 일이다. 다행히도 마이크로미터는 다양한 지표 수집 기능을 이미 만들어서 제공한다. 그리고 스프링부트 액츄에이터는 마이크로미터가 제공하는 지표 수집을 `@AutoConfiguration`을 통해 자동으로 등록해준다.

```json
// 20230429112232
// http://localhost:8081/actuator/metrics

{
  "names": [
    "application.ready.time",
    "application.started.time",
    "disk.free",
    "disk.total",
    "executor.active",
    "executor.completed",
    "executor.pool.core",
    "executor.pool.max",
    "executor.pool.size",
    "executor.queue.remaining",
    "executor.queued",
    "hikaricp.connections",
    "hikaricp.connections.acquire",
    "hikaricp.connections.active",
    "hikaricp.connections.creation",
    "hikaricp.connections.idle",
    "hikaricp.connections.max",
    "hikaricp.connections.min",
    "hikaricp.connections.pending",
    "hikaricp.connections.timeout",
    "hikaricp.connections.usage",
    "jdbc.connections.active",
    "jdbc.connections.idle",
    "jdbc.connections.max",
    "jdbc.connections.min",
    "jvm.buffer.count",
    "jvm.buffer.memory.used",
    "jvm.buffer.total.capacity",
    "jvm.classes.loaded",
    "jvm.classes.unloaded",
    "jvm.compilation.time",
    "jvm.gc.live.data.size",
    "jvm.gc.max.data.size",
    "jvm.gc.memory.allocated",
    "jvm.gc.memory.promoted",
    "jvm.gc.overhead",
    "jvm.gc.pause",
    "jvm.info",
    "jvm.memory.committed",
    "jvm.memory.max",
    "jvm.memory.usage.after.gc",
    "jvm.memory.used",
    "jvm.threads.daemon",
    "jvm.threads.live",
    "jvm.threads.peak",
    "jvm.threads.started",
    "jvm.threads.states",
    "logback.events",
    "process.cpu.usage",
    "process.files.max",
    "process.files.open",
    "process.start.time",
    "process.uptime",
    "system.cpu.count",
    "system.cpu.usage",
    "system.load.average.1m",
    "tomcat.sessions.active.current",
    "tomcat.sessions.active.max",
    "tomcat.sessions.alive.max",
    "tomcat.sessions.created",
    "tomcat.sessions.expired",
    "tomcat.sessions.rejected"
  ]
}
```

JVM Memory used

```json
// 20230429112538
// http://localhost:8081/actuator/metrics/jvm.memory.used

{
  "name": "jvm.memory.used",
  "description": "The amount of used memory",
  "baseUnit": "bytes",
  "measurements": [
    {
      "statistic": "VALUE",
      "value": 137419200
    }
  ],
  "availableTags": [
    {
      "tag": "area",
      "values": [
        "heap",
        "nonheap"
      ]
    },
    {
      "tag": "id",
      "values": [
        "CodeHeap 'profiled nmethods'",
        "G1 Old Gen",
        "CodeHeap 'non-profiled nmethods'",
        "G1 Survivor Space",
        "Compressed Class Space",
        "Metaspace",
        "G1 Eden Space",
        "CodeHeap 'non-nmethods'"
      ]
    }
  ]
}
```

availableTags ⇒ 필터를 걸어서 보는 방법. queryString으로 사용 가능

`area` tag의 `heap` value에 대한 조회

```json
// 20230429112700
// http://localhost:8081/actuator/metrics/jvm.memory.used?tag=area:heap

{
  "name": "jvm.memory.used",
  "description": "The amount of used memory",
  "baseUnit": "bytes",
  "measurements": [
    {
      "statistic": "VALUE",
      "value": 49812480
    }
  ],
  "availableTags": [
    {
      "tag": "id",
      "values": [
        "G1 Survivor Space",
        "G1 Old Gen",
        "G1 Eden Space"
      ]
    }
  ]
}
```

## 다양한 메트릭

JVM 메트릭

CPU 메트릭

애플리케이션 시작 메트릭

- `application.started.time` :  애플리케이션이 시작하는 데 걸리는 시간(`ApplicationStartedEvent`로 측정)
- `application.ready.time`: 애플리케이션이 요청을 처리할 준비가 되는 데 걸리는 시간(`ApplicationReadyEvent`로 측정)
- 스프링은 내부에 여러 초기화 단계가 있고 각 단계별로 내부에서 애플리케이션 이벤트를 발행
- `ApplicationStartedEvent` : 스프링 컨테이너가 완전히 실행된 상태이다. 이후에 커멘드 라인 러너가 호출
- `ApplicationReadyEvent`: 커맨드 라인 러너가 실행된 이후에 호출

스프링 MVC 메트릭

- 스프링 MVC 컨트롤러가 처리하는 모든 요청을 다룬다.
- 메트릭 이름: `http.server.requests`

데이터소스 메트릭

- `DataSource` 커넥션 풀에 관한 메트릭을 확인할 수 있다.
- 메트릭 이름: `jdbc.connections.` 으로 시작
- `hikaricp` 를 통해 더 많은 메트릭을 확인할 수 있다.

로그 메트릭

톰캣 메트릭

- 톰캣 메트릭을 확인하기 위해서는 외부 설정 파일을 통해 설정값 주입 필요

    ```yaml
    server:
    	tomcat:
    		mbeanregistry:
    			enabled: true
    ```


기타 여러 메트릭도 있고 `사용자 정의 메트릭`도 정의해서 쓸 수 있다.

기본적인 메트릭 수들을 만들어 놓고 핵심적인 비즈니스 메트릭을 추가하는 것이 좋다.

이렇게 많은 메트릭들을 어딘가에 지속해서 보관해야 과거의 데이터들도 확인할 수 있을 것이다. 따라서 이를 저장할 `데이터베이스`도 필요하고 확인할 수 있는 `대시보드`도 필요하다.

# 프로메테우스와 그라파나

> **참고**
프로메테우스와 그라파나는 그 내용을 다루는 책이 있을 정도로 내용이 방대하기 때문에 자세한 내용을 다루는 것은 강의 내용을 벗어난다. 여기선 각 기술들을 어떻게 다루고 활용하는 지 기초적인 내용과 올바른 방향을 설명하는 것에 초점을 맞추고 있다. 따라서 필요할 경우 추가적인 학습이 필요하다.
>

## 프로메테우스

애플리케이션에서 발생한 메트릭을 과거 이력까지 함께 확인하기 위해 메트릭을 보관하고 DB 역할

### 아키텍처
![image](https://user-images.githubusercontent.com/61923768/236794783-797cf4d8-c474-4257-8016-060f5b9db249.png)

프로메테우스 공식문서 > 소개 > 개요([https://prometheus.io/docs/introduction/overview/](https://prometheus.io/docs/introduction/overview/))

### 설치(Docker를 이용하기)

[awesome-compose/prometheus-grafana at master · docker/awesome-compose](https://github.com/docker/awesome-compose/tree/master/prometheus-grafana)

### 애플리케이션 설정

우리 애플리케이션을 연동에 메트릭을 수집하기 위해서 2가지 작업이 필요하다.

1. 우리 메트릭을 프로메테우스 포맷으로 출력할 수 있도록 만들기
2. 프로메테우스에서 주기적으로 메트릭을 가져가도록 만들기

어플리케이션 매트릭 **프로메테우스 포맷으로 출력하기**

- 스프링부트의 액츄에이터 포맷은 프로메테우스가 바로 이해할 수 없음
- 따라서 포맷을 변경해야 함 → 이 역할을 `마이크로미터`가 해줌
- 내부적으로 마이크로미터는 자체 표준방식으로 측정 중이기 떄문에 원하는 포맷으로 바꾸기 위한 구현체 선택만 하면 된다.
- 의존성 추가를 하면 자동으로 마이크로미터 프로메테우스 구현체를 등록해서 동작하도록 설정해준다.

    ```kotlin
    implementation("io.micrometer:micrometer-registry-prometheus:1.10.5")
    ```

  `/actuator` → promethues 추가

    ```json
    {
    	"_links": {
    		// ...
    		"prometheus": {
    		      "href": "http://localhost:8081/actuator/prometheus",
    		      "templated": false
    		    },
    		// ...
    	}
    }
    ```

  `/actuator/promethues` → 프로메테우스 포맷으로 출력된 데이터 보임

    ```json
    # HELP jvm_buffer_count_buffers An estimate of the number of buffers in the pool
    # TYPE jvm_buffer_count_buffers gauge
    jvm_buffer_count_buffers{id="mapped - 'non-volatile memory'",} 0.0
    jvm_buffer_count_buffers{id="mapped",} 0.0
    jvm_buffer_count_buffers{id="direct",} 10.0
    # HELP jvm_gc_pause_seconds Time spent in GC pause
    # TYPE jvm_gc_pause_seconds summary
    jvm_gc_pause_seconds_count{action="end of minor GC",cause="Metadata GC Threshold",gc="G1 Young Generation",} 1.0
    jvm_gc_pause_seconds_sum{action="end of minor GC",cause="Metadata GC Threshold",gc="G1 Young Generation",} 0.004
    ```


**프로메테우스 수집 설정**

프로메테우스 폴더에 있는 `prometheus.yml` 파일 수정

```yaml
scrape_configs:
#  스프링 어플리케이션 연동
  - job_name: spring-actuator
    metrics_path: /actuator/prometheus
    scrape_interval: 1s # 수집 주기는 테스트를 위한 1초 단위로 설정(10초 ~ 1분)
    static_configs:
      - targets: ['localhost:8081']
```

**게이지와 카운터**

- 메트릭을 크게 게이지와 카운터 2가지로 분류
- 게이지(Gauge): 오르고 내리는 값. 현재 상태를 그대로 출력
- 카운터(Counter): 오르기만 하는 값. 누적 상태를 출력
    - 누적으로 증가만 하기 때문에 원하는 것들 바로 확인하기가 어려움 → `increase()`, `rate()`
    - `increase()` → 증가값을 보여줌. 벡터와 함께 사용(단위시간)

        ```
        increase(http_server_requests_seconds_count{uri="/log"}[1m]
        ```

    - `rate()` → 증가율을 보여줌. 벡터와 함께 사용(단위시간)

        ```
        rate(http_server_requests_seconds_count{uri="/log"}[1m]
        ```


    ref: [https://prometheus.io/docs/prometheus/latest/querying/functions/](https://prometheus.io/docs/prometheus/latest/querying/functions/)


### 그라파나

프로메테우스가 DB라면, 이 DB에 있는 데이터를 불러서 사용자가 보기 편하게 보여주는 대시보드

구성

- 대시보드 > 판넬 > Query

**공유 대시보드**

[Dashboards | Grafana Labs](https://grafana.com/grafana/dashboards/)

이미 누가 만들어 놓은 대시보드의 형태를 가져와서 사용

- JVM(Micrometer): [https://grafana.com/grafana/dashboards/4701-jvm-micrometer/](https://grafana.com/grafana/dashboards/4701-jvm-micrometer/)
- SpringBoot System Monitor: [https://grafana.com/grafana/dashboards/11378-justai-system-monitor/](https://grafana.com/grafana/dashboards/11378-justai-system-monitor/)

실제 문제 확인방법

- 실무에서 주로 발생하는 문제 사례
    1. CPU 사용량 초과
    2. JVM 메모리 사용량 초과
    3. 커넥션 풀 고갈
    4. 에러 로그 급증

## 전체구조

1. 스프링부트 액츄에이터와 마이크로미터를 사용하면 수많은 메트릭을 자동으로 생성한다.
    1. 이때 마이크로미터의 프로메테우스 구현체가 프로메테우스가 읽을 수 있는 포맷으로 메트릭을 생성
2. 프로메테우스는 만들어진 메트릭을 수집, 저장한다.
3. 그라파나는 저장된 메트릭을 이용해 대시보드 툴을 구성하고 사용자에게 보여준다.
