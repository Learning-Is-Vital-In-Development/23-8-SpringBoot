### 1. 외부 설정 사용 - Environment     
#### 장점
- application.properties 에 필요한 외부 설정을 추가하고, Environment 를 통해서 해당 값들을    
읽어서, MyDataSource 를 만들었다.   
- 향후 외부 설정 방식이 달라져도, 예를 들어서 설정 데이터( application.properties )를 사용하다가     
커맨드 라인 옵션 인수나 자바 시스템 속성으로 변경해도 애플리케이션 코드를 그대로 유지할 수 있다.     
#### 단점     
- 이 방식의 단점은 Environment 를 직접 주입받고, env.getProperty(key) 를 통해서 값을 꺼내는    
과정을 반복해야 한다는 점이다. 스프링은 @Value 를 통해서 외부 설정값을 주입 받는 더욱 편리한 기능을    
제공한다.    
### 2. 외부설정 사용 - @Value
#### 장점    
- @Value 를 사용하면 외부 설정값을 편리하게 주입받을 수 있다.
- 참고로 @Value 도 내부에서는 Environment 를 사용한다.
- @Value 에 ${} 를 사용해서 외부 설정의 키 값을 주면 원하는 값을 주입 받을 수 있다.
- @Value 는 필드에 사용할 수도 있고, 파라미터에 사용할 수도 있다.
- myDataSource1() 은 필드에 주입 받은 설정값을 사용한다.
- myDataSource2() 는 파라미터를 통해서 설정 값을 주입 받는다.
#### 기본값     
- 만약 키를 찾지 못할 경우 코드에서 기본값을 사용하려면 다음과 같이 : 뒤에 기본값을 적어주면 된다.
예) @Value("${my.datasource.etc.max-connection:1}") : key 가 없는 경우 1 을 사용한다.
#### 정리
- application.properties 에 필요한 외부 설정을 추가하고, @Value 를 통해서 해당 값들을 읽어서,
MyDataSource 를 만들었다.
#### 단점   
- @Value 를 사용하는 방식도 좋지만, @Value 로 하나하나 외부 설정 정보의 키 값을 입력받고, 주입   
받아와야 하는 부분이 번거롭다. 그리고 설정 데이터를 보면 하나하나 분리되어 있는 것이 아니라 정보의     
묶음으로 되어 있다. 여기서는 my.datasource 부분으로 묶여있다. 이런 부분을 객체로 변환해서 사용할     
수 있다면 더 편리하고 더 좋을 것이다.    

### 3. 외부설정 사용 - @ConfigurationProperties
#### Type-safe Configuration Properties
- 스프링은 외부 설정의 묶음 정보를 객체로 변환하는 기능을 제공한다. 이것을 타입 안전한 설정 속성이라
한다.     
- 따라서 외부 설정을 자바 코드로 관리할 수 있는 것이다. 그리고 설정 정보 그 자체도 타입을 가지게 된다.
#### 정리
- application.properties 에 필요한 외부 설정을 추가     
- @ConfigurationProperties 를 통해서 MyDataSourcePropertiesV1 에 외부 설정의 값들을 설정    
그리고 해당 값들을 읽어서 MyDataSource 를 만들었다. 
### 4.외부설정 사용 - @ConfigurationProperties 생성자
- @ConfigurationProperties 는 Getter, Setter를 사용하는 자바빈 프로퍼티 방식이 아니라 생성자를
통해서 객체를 만드는 기능도 지원한다.
#### 정리
- application.properties 에 필요한 외부 설정을 추가하고, @ConfigurationProperties 의 생성자
주입을 통해서 값을 읽어들였다. Setter 가 없으므로 개발자가 중간에 실수로 값을 변경하는 문제가
발생하지 않는다.
### 5.외부설정 사용 - @ConfigurationProperties 검증
- @ConfigurationProperties 를 통해서 숫자가 들어가야 하는 부분에 문자가 입력되는 문제와 같은
타입이 맞지 않는 데이터를 입력하는 문제는 예방할 수 있다. 그런데 문제는 숫자의 범위라던가, 문자의 길이
같은 부분은 검증이 어렵다.
->개발자가 직접 하나하나 검증 코드를 작성해도 되지만, 자바에는 자바 빈 검증기(java bean validation)
이라는 훌륭한 표준 검증기가 제공된다.
#### 정리
- ConfigurationProperties 덕분에 타입 안전하고, 또 매우 편리하게 외부 설정을 사용할 수 있다. 그리고
검증기 덕분에 쉽고 편리하게 설정 정보를 검증할 수 있다.    
- 가장 좋은 예외는 컴파일 예외, 그리고 애플리케이션 로딩 시점에 발생하는 예외이다. 가장 나쁜 예외는 고객    
서비스 중에 발생하는 런타임 예외이다.
#### ConfigurationProperties 장점  
- 외부 설정을 객체로 편리하게 변환해서 사용할 수 있다.    
- 외부 설정의 계층을 객체로 편리하게 표현할 수 있다.     
- 외부 설정을 타입 안전하게 사용할 수 있다.    
- 검증기를 적용할 수 있다.    
### 6. YAML
- 스프링은 설정 데이터를 사용할 때 application.properties 뿐만 아니라 application.yml 이라는
형식도 지원한다.    
- YAML(YAML Ain't Markup Language)은 사람이 읽기 좋은 데이터 구조를 목표로 한다. 확장자는   
yaml , yml 이다. 주로 yml 을 사용한다.   
- application.properties 예시
```
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```
- application.yml 예시
```
environments:
 dev:
 url: "https://dev.example.com"
 name: "Developer Setup"
 prod:
 url: "https://another.example.com"
 name: "My Cool App"
```
- YAML의 가장 큰 특징은 사람이 읽기 좋게 계층 구조를 이룬다는 점이다.
- YAML은 space (공백)로 계층 구조를 만든다. space 는 1칸을 사용해도 되는데, 보통 2칸을 사용한다. 
- 일관성있게 사용하지 않으면 읽기 어렵거나 구조가 깨질 수 있다.
- 구분 기호로 : 를 사용한다. 만약 값이 있다면 이렇게 key: value : 이후에 공백을 하나 넣고 값을
넣어주면 된다.
### 7. PROFILE
#### 정리
- @Profile 을 사용하면 각 환경 별로 외부 설정 값을 분리하는 것을 넘어서, 등록되는 스프링 빈도 분리할 수
있다.
