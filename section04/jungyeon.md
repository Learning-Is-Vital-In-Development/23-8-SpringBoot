# Section 4. 스프링 부트 스타터와 라이브러리 관리
# 0.외부 라이브러리 버전 관리
- 버전 관리 기능을 사용하려면 io.spring.dependency-management 플러그인을 사용.
- http://localhost:8080/hello-spring 호출해서 정상 동작 확인
- 버전 정보 bom    
    https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/springboot-dependencies/build.gradle   
해당 build.gradle 문서안에 보면 bom 이라는 항목이 있다.   
각각의 라이브러리에 대한 버전이 명시되어 있는 것을 확인할 수 있다.   
물론 현재 프로젝트에서 지정한 스프링 부트 버전을 참고한다.   
- id 'org.springframework.boot' version '3.0.2' : 여기에 저징된 스프링 부트 버전 참고   
스프링 부트의 버전을 변경해보면 나머지 라이브러리들의 버전도 변하는 것을 확인할 수 있다.   

# 1.스프링 부트 스타터 - 자주 사용하는 것 위주
- spring-boot-starter : 핵심 스타터, 자동 구성, 로깅, YAML
- spring-boot-starter-jdbc : JDBC, HikariCP 커넥션풀
- spring-boot-starter-data-jpa : 스프링 데이터 JPA, 하이버네이트
- spring-boot-starter-data-mongodb : 스프링 데이터 몽고
- spring-boot-starter-data-redis : 스프링 데이터 Redis, Lettuce 클라이언트
- spring-boot-starter-thymeleaf : 타임리프 뷰와 웹 MVC
- spring-boot-starter-web : 웹 구축을 위한 스타터, RESTful, 스프링 MVC, 내장 톰캣
- spring-boot-starter-validation : 자바 빈 검증기(하이버네이트 Validator)
- spring-boot-starter-batch : 스프링 배치를 위한 스타터
