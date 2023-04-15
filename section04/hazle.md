
# 4. 스프링 부트 스타터와 라이브러리 관리 

스프링 부트는 라이브러리들을 편리하게 사용할 수 있는 다양한 기능들을 제공한다. 

- 외부 라이브러리 버전 관리
- 스프링 부트 스타터 제공

# 스프링 부트 라이브러리 버전 관리

스프링부트를 사용하면 개발자는 라이브러리 버전을 고르고  버전을 생략해도된다. 스프링 부트가 버전에 맞는 최적화 버전을 선택해준다. 

이 기능을 사용하려면 `io.spring.dependency-management`를 추가해줘야한다. 

### 버전 정보 bom

[spring-boot/build.gradle at main · spring-projects/spring-boot](https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-dependencies/build.gradle)

해당 build.gradle 문서를 보면 bom이라는 항목이 있고, 각각의 라이브러리에 대한 버전이 명시되어있는걸 확인할 수 있다. 

또한 현재 프로젝트에서 지정한 스프링 부트 버전을 참고한다.   `id 'org.springframework.boot' version '3.0.2’`


+) 스프링 부트가 관리하는 외부 라이브러리 버전 확인하는 방법

[Dependency Versions](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions.coordinates)


#  스프링 부트 스타터

스프링 부트는 이런 문제를 해결하기 위해 프로젝트를 시작하는데 필요한 관련 라이브러리를 모아둔 스프링 부트 스타터를 제공한다.

```java
dependencies {
 implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

- 스프링과 웹을 사용하고 싶으면 `spring-boot-starter-web`
스프링 웹 MVC, 내장 톰캣, JSON 처리, 스프링 부트 관련, LOG, YML 등등
- 스프링과 JPA를 사용하고 싶으면 `spring-boot-starter-data-jpa`
스프링 데이터 JPA, 하이버네이트 등등



## 스프링 부트 스타터 **-** 자주 사용하는 것 위주

`spring-boot-starter` : 핵심 스타터, 자동 구성, 로깅, YAML

`spring-boot-starter-jdbc` : JDBC, HikariCP 커넥션풀

`spring-boot-starter-data-jpa` : 스프링 데이터 JPA, 하이버네이트

`spring-boot-starter-data-mongodb` : 스프링 데이터 몽고

`spring-boot-starter-data-redis` : 스프링 데이터 Redis, Lettuce 클라이언트

`spring-boot-starter-thymeleaf` : 타임리프 뷰와 웹 MVC

`spring-boot-starter-web` : 웹 구축을 위한 스타터, RESTful, 스프링 MVC, 내장 톰캣

`spring-boot-starter-validation` : 자바 빈 검증기(하이버네이트 Validator)

`spring-boot-starter-batch` : 스프링 배치를 위한 스타터

+) 스프링 스타터 목록 확인 링크
[Developing with Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters)