# Section 10. 모니터링 메트릭 등록

## 메트릭 등록 - 예제 만들기

일반적인 메트릭은 다 제공되고 구성되어 있어서 가져다가 쓰면 된다.

우리 비즈니스에 맞는 메트릭들을 구성하고 만들고 싶을 때 커스텀 매트릭스를 만든다.

### **메트릭 정의하기**

주문수, 취소수

- 주문하면 주문 수가 증가한다. 취소하면 주문 수는 유지되며 취소 수가 증가한다.

재고 수량

- 상문을 주문하면 재고 수량이 감소한다.
- 상품을 취소하면 재고 수량이 늘어난다.
- 재고 물량이 들어오면 재고 수량이 늘어난다.

### 메트릭 등록하기

마이크로미터를 이용해 메트릭을 직접 등록하고 다른 곳에서 사용할 수 있도록 합니다.

### MeterRegistry

마이크로미터 기능을 제공하는 핵심 컴포넌트로 스프링으로 주입 받아서 사용하고 카운터, 게이지 등을 등록한다.

```kotlin
class OrderServiceV1(
    private val meterRegistry: MeterRegistry,
) : OrderService {
    override fun order() {
				// ...

        Counter.builder("my order").apply {
            tag("class", this.javaClass.name)
            tag("method", "order")
            description("order")
        }.register(meterRegistry).increment()
    }
}
```

`http://localhost:8081/actuator/metrics/my.order`

```json
{
  "name": "my.order",
  "description": "order",
  "measurements": [
    {
      "statistic": "COUNT",
      "value": 1.0
    }
  ],
  "availableTags": [
    {
      "tag": "method",
      "values": [
        "order"
      ]
    },
    {
      "tag": "class",
      "values": [
        "com.dojinyou.inflearn.roadmap.w6.order.v1.OrderServiceV1"
      ]
    }
  ]
}
```

프로메테우스 메트릭 확인하기

`http://localhost:8081/actuator/prometheus`

```json
# HELP my_order_total order
# TYPE my_order_total counter
my_order_total{class="com.dojinyou.inflearn.roadmap.w6.order.v1.OrderServiceV1",method="order",} 1.0
```

### @Counter(AOP)로 메트릭 등록하기

`OrderConfigV2`

```kotlin
@Configuration
class OrderConfigV2 {
		// CountedAspect가 등록되지 않으면 AOP 어노테이션이 동작하지 않음!!
    @Bean
    fun countedAspect(registry: MeterRegistry): CountedAspect {
        return CountedAspect(registry)
    }
}
```

`OrderServiceV2`

```kotlin
@Service // AOP를 위한 all open 플러그인을 적용 받기 위해서 자동 빈 등록을 이용
class OrderServiceV2 : OrderService {
    @Counted("my.order")
    override fun order() {}

    @Counted("my.order")
    override fun cancel() {}

    override fun getStock(): AtomicInteger {}
}
```

### Timer 메트릭 등록하기

Timer는 실행시간을 함께 측정해준다.

- `seconds_count`: 누적 실행 수 - `counter`
- `seconds_sum`: 실행시간의 합 - `sum`
- `seconds_max`: 최대 실행시간 - `guage`
    - 내부에 타임 윈도우가 존재하여 1~3분 마다 최대 실행시간이 다시 계산된다.
- `seconds_sum/seconds_count`: 평균실행시간(계산을 통해 구할 수 있다.)

`OrderServiceV3`

```kotlin
class OrderServiceV3(
    private val meterRegistry: MeterRegistry,
) : OrderService {
    override fun order() {
        val timer = Timer.builder("my.order")
            .tag("class", this.javaClass.name)
            .tag("method", "order")
            .description("order")
            .register(meterRegistry)

        timer.record(
            IntSupplier {
                logger.info("주문")
                stock.getAndDecrement()
            },
        )
    }

    override fun cancel() {
        val timer = Timer.builder("my.order")
            .tag("class", this.javaClass.name)
            .tag("method", "cancel")
            .description("cancel")
            .register(meterRegistry)

        timer.record(
            IntSupplier {
                logger.info("취소")
                stock.getAndIncrement()
            },
        )
    }

    override fun getStock(): AtomicInteger {}
}
```

`http://localhost:8081/actuator/prometheus`

```
# HELP my_order_seconds order
# TYPE my_order_seconds summary
my_order_seconds_count{class="com.dojinyou.inflearn.roadmap.w6.order.v3.OrderServiceV3",method="cancel",} 8.0
my_order_seconds_sum{class="com.dojinyou.inflearn.roadmap.w6.order.v3.OrderServiceV3",method="cancel",} 0.001340918
my_order_seconds_count{class="com.dojinyou.inflearn.roadmap.w6.order.v3.OrderServiceV3",method="order",} 6.0
my_order_seconds_sum{class="com.dojinyou.inflearn.roadmap.w6.order.v3.OrderServiceV3",method="order",} 0.001466541
# HELP my_order_seconds_max order
# TYPE my_order_seconds_max gauge
my_order_seconds_max{class="com.dojinyou.inflearn.roadmap.w6.order.v3.OrderServiceV3",method="cancel",} 3.46834E-4
my_order_seconds_max{class="com.dojinyou.inflearn.roadmap.w6.order.v3.OrderServiceV3",method="order",} 6.68709E-4
```

```kotlin
@Service
@Timed("my.order")
class OrderServiceV4 : OrderService {}
```

### 게이지 메트릭 등록하기

게이지는 임의의 오르고 내릴 수 있는 값으로 현재의 상태를 확인하는 데 사용

```kotlin
@Configuration
class MyStockConfigV2 {

    @Bean
    fun stockSize(orderService: OrderService): MeterBinder {
        return MeterBinder {
                registry ->
            run {
                Gauge.builder("my.stock") {
                    orderService.getStock().get()
                }.register(registry)
            }
        }
    }
}
```

## 실무 모니터링 환경 구성 팁

### 모니터링 3단계

1. 대시보드
2. 애플리케이션 추적 - 핀포인트
3. 로그

### 대시보드

전체 시스템을 한 눈에 볼 수 있는 뷰

제품

- 마이크로미터, 프로메테우스, 그라파나

모니터링 대상

- 시스템 메트릭(CPU, 몌모리)
- 애플리케이션 메트릭(톰캣 쓰레드 풀, DB 커넥션 풀, 애플리케이션 호출 수)
- 비즈니스 메트릭(주문, 취소)

### 애플리케이션 추적

주로 http 요청을 추적, 마이크로서비스 환경에서는 분산 추적

제품

- 핀포인트(오픈소스), 스카우트(오픈소스), 와탭(상용), 제니퍼(상용)

### 로그

가장 디테일한 추적, 원하는 대로 커스텀 가능

같은 HTTP 요청에 대해서 묶어서 확인할 수 있는 방법 필요(`중요!`), MDC 적용

**파일로 직접 로그를 남기는 경우**

- 일반 로그와 에러 로그는 구분해서 남기자. → 에러 로그만 확인을 해서 문제를 바로 정리할 수 있음.

**클라우드에 로그를 저장하는 경우**

- 검색이 잘 되도록 구분하자.
