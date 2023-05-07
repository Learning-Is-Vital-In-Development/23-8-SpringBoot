# Section7: 외부 설정과 프로필2

**외부 설정**

- 설정 데이터(applications.properties)
- OS 환경변수
- 자바 시스템 속성
- 커맨드 라인 옵션 인수

**스프링이 제공하는 다양한 외부 설정 조회 방법**

- `Environment`
- `@Value` - 값 주입
- `@ConfigurationProperties` - 타입 안전한 설정 속성

## Environmet 방식

스프링 `Enviornment`의 `getProperty(key)`를 통해 값을 꺼내 설정 데이터를 사용할 수 있다.

이 방식의 단점은 `Environment`를 직접 주입 받고, `getProperty(key)`를 통해 값을 꺼내는 과정을 반복해야 한다.

## @Value

`@Value`를 사용하면 외부 설정값을 편리하게 주입받을 수 있다.

- `@Value`에 `${}`를 사용해서 외부설정의 키 값을 주면 원하는 값을 주입받을 수 있다.
- `@Value` 는 필드에 사용할 수도 있고, 파라미터에 사용할 수도 있다.
- 만약 키를 찾지 못할 경우 코드에서 기본값을 사용하려면 : 뒤에 기본값을 적어주면 된다.

**단점**

`@Value` 로 하나하나 외부 설정 정보의 키 값을 입력받고, 주입 받아와야 하는 부분이 번거롭다.

## @ConfigurationProperties

### **Type-safe Configuration Properties**

스프링은 외부 설정의 묶음 정보를 객체로 변환하는 기능을 제공하는데, 이를 **타입 안전한 설정 속성**이라고 한다.

→ 외부 설정을 자바 코드로 관리할 수 있고, 설정 정보 그 자체도 타입을 가지게 된다.

기본적인 사용법은  외부 설정을 주입 받을 객체를 생성해 각 필드를 외부 설정의 키 값에 맞춰 준비한다.

`@ConfigurationProperties`가 있으면 외부 설정을 주입받는 객체라는 뜻이며, KEY의 묶음 시작점을 적어준다. ex) `@ConfigurationProperties("my.datasource")`

기본 주입 방식은 자바빈 프로퍼티 방식이다.

`ConfigurationProperties` 를 사용하면 타입 안전한 설정 속성을 사용할 수 있기 때문에 타입 안전한 설정 속성이라고 하며, `ConfigurationProperties`로 만든 데이터는 타입에 대해 믿고 사용할 수 있다.

자바빈 프로퍼티 방식으로 주입하니 `Setter` 메서드를 가지고 있기 때문에 누군가 실수로 값을 변경하는 문제가 발생할 수 있다. 이 값들은 외부 설정값을 사용해 초기에 설정 되고, 이후 변경하면 안된다.

`Setter`를 제거해야 한다.

### @ConfigurationProperties 생성자

- 생성자를 만들어 생성자를 통해 설정 정보를 주입
- `@DefualtValue`: 해당 값을 찾을 수 없는 경우 기본 값 사용

<aside>
💡 부트 3.0 이전에는 생성자 바인딩 시 `@ConsturctorBinding` 어노테이션 필수 사용

</aside>

### @ConfigurationProperties 검증

`@ConfigurationProperties` 은 자바 객체이기 때문에 스프링이 자바 빈 검증기를 사용할 수 있도록 지원한다.

**build.gradle - 의존성 추가**

```groovy
implementation 'org.springframework.boot:spring-boot-starter-validation' //추가
```

- `@NotEmpty`: 필수 값
- `@Min`, `@Max` : 최대, 최소 값
- `@DurationMin`, `@DurationMax`: 최소 초, 최대 초

## YAML

스프링은 설정 데이터를 사용할 때 `application.properties` 뿐만 아니라 `application.yml` 이라는 형식도 지원한다.

- YAML의 가장 큰 특징은 사람이 읽기 좋게 계층 구조를 이룬다는 점이다.
- YAML은 `space` (공백)로 계층 구조를 만든다. `space` 는 1칸을 사용해도 되는데, 보통 2칸을 사용한다.
- 일관성있게 사용하지 않으면 읽기 어렵거나 구조가 깨질 수 있다.
- 구분기호로 `:`를 사용한다.

<aside>
💡 `application.properties`, `application.yml` 을 같이 사용하면 `application.properties` 가 우선권을 가진다.

</aside>

## @Profile

`@Profile` 어노테이션을 사용하면 해당 프로필이 활성화된 경우에만 빈을 등록한다.

스프링은 `@Conditional` 기능을 활용해서 개발자가 더 편리하게 사용할 수 있는 @Profile 기능을 제공한다.

`@Profile` 을 사용하면 각 환경 별로 외부 설정 값을 분리하는 것을 넘어서, 등록되는 스프링 빈도 분리할 수 있다.
