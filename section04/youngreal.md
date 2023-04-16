프로젝트 시작시 어떤 라이브러리를 선택해야할지, 어떤버전을 선택해야할지, 라이브러리간의 호환이 어떨지 등을 고려해야한다.

### 스프링부트의 라이브러리 버전관리

```java
plugins {
	... 
    
    id 'io.spring.dependency-management' version '1.0.15.RELEASE'
}
```
- io.spring.dependency-management 이녀석이 있다면 버전을 명시해주지 않아도 된다. 버전을 관리해준다. 

- spring-boot-dependencies 의 라이브러리의 버전들이 직접 명시되어있는 bom이라는 정보를 참고해서 부트버전에 맞는 라이브러리의 버전을 참고하게된다.

- 대중적이지 않은 라이브러리들은 버전직접 명시해야한다. 


### 스프링 부트 스타터

- 스프링 웹MVC, 내장톰캣, JSON, LOG, YML 등 대중적인 다양한 라이브러리들을 모아두었다.

- `ext['tomcat.version'] = '10.1.4' 와같이 외부라이브러리의 버전도 필요하다면 변경할수있다. 
