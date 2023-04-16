라이브러리 관리의 어려움

- 프로젝트를 처음 시작할 때 라이브러리 사용을 고민하고 선택해야 한다.
- 각 라이브러리의 버전과 호환을 고민해야했고 처음 프로젝트 세팅에 많은 시간을 소비했다.

스프링부트는 개발자가 라이브러리를 편리하게 사용할 수 있는 다양한 기능을 제공

- 외부 라이브러리 버전 관리
- 스프링 부트 스타터 제공

# 라이브러리 직접 관리

- 라이브러리를 직접 고르는 방법

`build.gradle`

```java
plugins {
    id 'org.springframework.boot' version '3.0.2'
    //id 'io.spring.dependency-management' version '1.1.0'
    id 'java'
}
```

- `io.spring.dependency-management 사용 해제`

```java
dependencies {

    //1. 라이브러리 직접 지정
    //스프링 웹 MVC
    implementation 'org.springframework:spring-webmvc:6.0.4'
    //내장 톰캣
    implementation 'org.apache.tomcat.embed:tomcat-embed-core:10.1.5'
    //JSON 처리
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.14.1'
    //스프링 부트 관련
    implementation 'org.springframework.boot:spring-boot:3.0.2'
    implementation 'org.springframework.boot:spring-boot-autoconfigure:3.0.2'
    //LOG 관련
    implementation 'ch.qos.logback:logback-classic:1.4.5'
    implementation 'org.apache.logging.log4j:log4j-to-slf4j:2.19.0'
    implementation 'org.slf4j:jul-to-slf4j:2.0.6'
    //YML 관련
    implementation 'org.yaml:snakeyaml:1.33'
}
```

- 각 라이브러리와 버전을 직접 지정해줘야한다.

라이브러리 직접 선택시 발생하는 문제

- 웹 프로젝트를 하기 위해 수많은 라이브러리를 알아야 한다.
- 각 라이브러리의 버전을 골라서 선택해야한다.
- **각 라이브러리의 호환성을 신경써야 했다.**

# 스프링 부트 라이브러리 버전 관리

스프링 부트는 개발자 대신에 라이브러리 버전을 직접 관리한다.

```java
plugins {
    id 'org.springframework.boot' version '3.0.2'
    id 'io.spring.dependency-management' version '1.1.0'
    id 'java'
}
```

- `io.spring.dependency-management` 를 사용해준다.

```java
dependencies {
/*
    //2. 스프링 부트 라이브러리 버전 관리
    //스프링 웹, MVC
    implementation 'org.springframework:spring-webmvc'
    //내장 톰캣
    implementation 'org.apache.tomcat.embed:tomcat-embed-core'
    //JSON 처리
    implementation 'com.fasterxml.jackson.core:jackson-databind'
    //스프링 부트 관련
    implementation 'org.springframework.boot:spring-boot'
    implementation 'org.springframework.boot:spring-boot-autoconfigure'
    //LOG 관련
    implementation 'ch.qos.logback:logback-classic'
    implementation 'org.apache.logging.log4j:log4j-to-slf4j'
    implementation 'org.slf4j:jul-to-slf4j'
    //YML 관련
    implementation 'org.yaml:snakeyaml'
*/
}
```

- 버전 정보를 빼고 사용해도 사용할 수 있다.

[spring-boot/build.gradle at main · spring-projects/spring-boot](https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-dependencies/build.gradle)

- 위 라이브러리에서 버전을 관리한다.

![image](https://user-images.githubusercontent.com/66561524/232308340-b98f33e4-0857-486b-bfe1-a5e93199c650.png)

- bom 이라는 항목이 있고 각각의 라이브러리에 대한 버전이 명시되어있다.
- 현재 프로젝트에서 지정한 스프링 부트 버전을 참고한다.
    - `id 'org.springframework.boot' version '3.0.2'`
    - 스프링 부트의 버전을 변경해보면 나머지 라이브러리들의 버전도 변하는 것을 확인할 수 있다.

참고 - BOM(Bill of materials)

자재 명세서(Bill of materials)란 제품구성하는 모든 부품들에 대한 목록이다. 부품이 복잡한 요소들로 구성된 조립품인 경우는 계층적인 구조로 작성될 수 있다.

참고 - 스프링 부트가 관리하는 외부 라이브러리 버전을 확인하는 방법

[Dependency Versions](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html)

- 현재 스프링부트 버전에 따른 라이브러리의 버전을 확인할 수 있다.

참고 - 스프링 부트가 관리하지 않는 라이브러리

스프링 부트가 관리하지 않는 라이브러리는 직접 버전을 명시해줘야 한다.

- 스프링 부트는 스프링 자신을 포함해서 수많은 외부 라이브러리의 버전을 최적화해서 관리
- 해당 버전은 스프링 부트그ㅏ 각 라이브러리의 호환성을 테스트했기 때문에 안전하게 사용할 수 있다.

# 스프링 부트 스타터

- 웹 프로젝트를 실행하려면 수많은 라이브러리가 필요하다.
- 스프링 부트는 이런 문제를 해결하기 위해 프로젝트를 시작하는데 필요한 관련 라이브러리를 모아둔 스프링 부트 스타터를 제공한다.

```java
dependencies {
    //3. 스프링 부트 스타터
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

- 이 의존성 하나로 지금까지 우리가 직접 넣어주었던 모든 라이브러리가 포함된다.
- 사용하기 편리하게 의존성을 모아둔 세트이다.
    - 스타터도 스타터를 사용할 수 있다.
- 웹을 사용하고 싶으면 `spring-boot-starter-web`
- JPA를 사용하고 싶으면 `spring-boot-starter-data-jpa`

![image](https://user-images.githubusercontent.com/66561524/232308377-54159e90-f7a7-41ab-9f45-2519f6fd0968.png)


스프링부트 스타터 - 이름 패턴

- `spring-boot-starter-*`
- 비공식 스타터: `thirdpartyproject-spring-boot-starter-*`
    - `mybatis-spring-boot-starter`
    
많은 starters 들이 있다. spring initializer 에서 쉽게 찾을 수 있다.

[Developing with Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters)

**라이브러리 버전 변경**

외부 라이브러리의 버전을 변경하고 싶을 때 다음과 같은 형식을 사용하면 된다.

```java
//스프링 부트 외부 라이브러리 버전 변경
ext['tomcat.version']='10.1.4'
```

[Dependency Versions](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions.properties)

앞의 외부 라이브러리 이름을 여기서 확인한다.

![image](https://user-images.githubusercontent.com/66561524/232308402-df78d058-183a-4340-954c-a218ce23cec4.png)
