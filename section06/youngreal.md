## 외부설정
애플리케이션을 여러 다른 환경에서 사용해야할때가 있으며, 환경에따라 서로 다른 설정값이 존재할수있다.


과거..
dev, prod 환경에 맞게 애플리케이션을 배포하고 빌드했다. 


현재.. 
빌드는 한번만하고 환경에맞춰 외부 설정값을 주입하는방식으로 환경에 따라 변하는 설정값을 실행시점에 주입한다. 

변하는것(외부 설정값)과 변하지않는것(애플리케이션 코드, 빌드 결과물)을 분리하여 유연한 방식을 사용한다

### 외부설정 4가지 방식

- OS 환경변수 
  - 자바프로그램에서만 사용하는것이 아닌 OS 전역에서 사용할수있다는 부분이 불편할수있다.
 
- 자바 시스템 속성
  - 실행 JVM 안에서 접근 가능한 외부설정
  
  - 자바 프로그램 실행시 속성을 줄수있다.
  **java -Durl=dev -jar application.jar**
  
  - 코드 안에서 사용할수있으며, 외부로 설정을 분리하는 효과는 없다. 개발서버, 운영서버에서 각각 다른 jar옵션을 줘서 실행할수있다.

- 자바 커맨드라인 인수
  - **java -jar application.jar A B**
  
  - key, value 형식이 아닌 단순 문자열이기때문에 key,value 형태로 사용하려면 개발자가 직접 파싱해서 값을 집어넣어야한다. 
  
  -  커맨드 라인 옵션 인수(스프링의 제공)
     - **--url=devdb** 와같이 대시 2개로 옵션인수로 구분해 key,value형태로 사용할수있게 해준다. 
  
- 외부파일 (설정 데이터) 
  - 애플리케이션에서 특정 위치의 파일을 읽게끔 해서  설정정보를 읽히게끔 하는것 

#### 커맨드라인 옵션인수와 스프링부트

스프링부트 내부적으로  ApplicationArguments를 빈으로 등록해두고 입력받은 커맨드 라인을 저장해둔다. 


### 외부설정 - 스프링통합
환경변수의 종류를 변경했을때 애플리케이션의 코드를 수정하지않도록 유연하게 사용할수있도록 스프링은 Environment 와 PropertySource로 추상화 해두었다.

![](https://velog.velcdn.com/images/rodlsdyd/post/16cdb7fd-ffe5-4347-b133-8ee596fd634e/image.png)


#### Environment

- 특정 외부설정에 종속되지않고 key=value 형식의 외부설정에 접근할수있다. 

- 외부 설정 방식을 변경하더라도 코드변경이 필요없다. 

- 모든 외부설정은 Environment를 통해 조회하면된다.

#### PropertySource

- 로딩시점에 필요한 PropertySource들을 생성해 Environment에서 사용할수있게 연결해둔다. 


### 설정데이터 1 - 외부 파일

환경변수가 엄청 많아진다면 프로그램 실행시마다 입력하기 번거롭고 관리도 어려워진다.

대안으로 파일에 저장해서 관리하는 방법이있다.

- 프로그램 외부에 properties 파일을 만들어서 설정값들을 저장해두고 애플리케이션 로딩 시점에 해당파일을 읽게끔 하는 방법 

- 운영, 개발환경에서 각각 다른 properties 파일을 관리해서 사용할수있다.

- 그러나, 서버가 10대라면 10개의 properties 파일을 전부 관리해야한다. 

- 설정값의 변경이력을 확인하기 어렵다. 설정값의 변경과 코드의 변경의 이력을 확인하기 어렵다. 

### 설정데이터 2 - 내부 파일 분리 

- 프로젝트안에 설정 데이터도 함께 포함해서 관리하는방법 
  - 개발용이면 application-dev.properties, 운영용이면 application-prod.properties  

- 개발용이 필요하면 dev 프로필, 운영용이 필요하면 prod 프로필을 줘서 설정정보를 배포하면 된다.

### 설정데이터 3- 내부 파일 합체

- ---(yml)나 #--- 로 properties 하나로 구분할수있다.
  
### 우선순위 - 설정 데이터

- spring.config.activate.on-profile 같은 설정이 특별히 없는 properties 내용들은 프로필 지정과 무관하게 항상 사용된다.

- 스프링은 설정파일의 위에서부터 읽으면서 값을 설정하고 특정 프로필이 설정되있으면 해당 프로필과 일치하는 데이터를 읽어서 덮어쓴다.

- 값이 순서대로 읽고 덮어쓰는것에 유의해야한다. 


### 우선순위 - 전체 

> 더 유연하고, 범위가 좁은것이 우선권을 가진다. 

1. 설정데이터(application.properties)
 - 1. 내부 jar application.properties
 - 2. jar 내부 프로필 적용파일
 - 3. jar 외부 application.properties
 - 4. jar 외부 프로필 적용파일
 
2. OS 환경변수
3. 자바 시스템 속성
4. 커맨드 라인 옵션인수
5. @TestPropertySource(테스트 케이스 실행시)
