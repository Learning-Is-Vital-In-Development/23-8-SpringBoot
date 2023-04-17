# Section 4. 스프링 부트 스타터와 라이브러리 관리

### **라이브러리 관리의 어려움**

- 다양한 라이브러리 선택지 중 어떤 것을 선택 해야할 지 어려움.
- 라이브러리의 버전 선택
- 버전간 호환성 고려

## 스프링부트 라이브러리 버전 관리

스프링 부트는 개발자 대신 버전을 직접 관리해줌. 현재의 스프링부트 버전에 맞는 라이브러리의 버전을 자동으로 선택 됨.

→ `io.spring.dependency-management`  플로그인이 `spring-boot-depedencies` 에 있는 bom 정보를 참고한다.

### ***버전 정보 bom**

- [https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-dependencies/build.gradle](https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-dependencies/build.gradle)
- 위 문서 안에 `bom` 블럭이 있고 해당 블럭 안에 라이브러리에 대한 버전이 기재되어 있다.

버전 관리를 하지 않는 라이브러리의 경우에는 버전을 명시적으로 명시해주어야 한다.

## 스프링부트 스타터

- 스프링부트는 웹 개발이 필요한 라이브러리를 모아서 `org.springframework.boot:spring-boot-starter-web` 로 제공한다.
- 사용자는 스타터 라이브러리 하나로 사용 가능하다.

### **주요 스프링 부트 스타터(자주 사용하는 것 위주)**

- `spring-boot-starter` : 핵심 스타터, 자동 구성, 로깅, YAML
- `spring-boot-starter-jdbc` : JDBC, HikariCP 커넥션풀
- `spring-boot-starter-data-jpa` : 스프링 데이터 JPA, 하이버네이트
- `spring-boot-starter-data-mongodb` : 스프링 데이터 몽고
- `spring-boot-starter-data-redis` : 스프링 데이터  redis, Lettuce 클라이언트
- `spring-boot-starter-web` : 웹구축위한 스타터, RESTful, 스프링 MVC, 내장 톰캣
- `spring-boot-starter-validation` : 자바 빈 검증기(hibernate validator)
- `spring-boot-starter-batch` : 스프링 배치를 위한 스타터
