# Section3: 스프링 부트 스타터와 라이브러리 관리

## 스프링 부트 스타터와 라이브러리 관리

**라이브러리 관리의 어려움**

프로젝트 시작 시 사용할 라이브러리를 고민하고 선택해야 한다. 추가로 각 라이브러리의 버전까지 고민해야 한다. 심지어는 각 라이브러리의 버전들끼리의 호환성도 고려해야 했다. 이러한 문제들 때문에 프로젝트 초기 세팅 시간이 상당히 많이 소비 되었었다.

스프링 부트는 개발자가 라이브러리 관리를 편하게 할 수 있는 기능을 제공한다.

- 외부 라이브러리 버전 관리
- 스프링 부트 스타터 제공

### 스프링 부트 라이브러리 버전 관리

스프링 부트가 제공하는 버전 관리 기능을 사용하려면 `io.spring.dependency-management` 플러그인을 사용하자.

[**스프링 부트가 관리하는 외부 라이브러리 버전 확인**](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions.coordinates)

**스프링부트가 관리하지 않는 라이브러리를 사용할 때는 라이브러리 버전을 직접 적어주어야 한다.**

### 스프링 부트 스타터

- 웹 프로젝트 하나에 수 많은 라이브러리가 필요하다.
- 스프링 부트는 이런 프로젝트를 시작하는데 필요한 관련 라이브러리를 모아둔 스프링 부트 스타터를 제공한다.

```groovy
dependencies {
      implementation 'org.springframework.boot:spring-boot-starter-web'
  }
```

**스프링 부트 스타터 - 이름 패턴**

- `spring-boot-starter-*`
- 쉽게 찾게 도와줌
- 공식: `spring-boot-starter-*`
- 비공식: `thirdpartyproject-spring-boot-starter`
    - ex) `mybatis-spring-boot-starter`
