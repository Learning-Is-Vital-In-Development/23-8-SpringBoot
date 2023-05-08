### 8. Actuator
- 스프링 부트가 제공하는 액추에이터는 이런 프로덕션 준비 기능을 매우 편리하게 사용할 수 있는 다양한
편의 기능들을 제공한다. 
- 더 나아가서 마이크로미터, 프로메테우스, 그라파나 같은 최근 유행하는 모니터링
시스템과 매우 쉽게 연동할 수 있는 기능도 제공한다.
- 참고로 액추에이터는 시스템을 움직이거나 제어하는 데 쓰이는 기계 장치라는 뜻이다.
#### 액츄에이터 시작
- 액츄에이터가 제공하는 프로덕션 준비 기능을 사용하려면 스프링 부트 액츄에이터 라이브러리를 추가해야
한다.
- 액츄에이터는 헬스 상태 뿐만 아니라 수
많은 기능을 제공하는데, 이런 기능이 웹 환경에서 보이도록 노출해야 한다.
#### 엔드포인트 설정
- 엔드포인트를 사용하려면 다음 2가지 과정이 모두 필요하다.
1. 엔드포인트 활성화     
해당 기능 자체를 사용할지 말지 on , off 를 선택.
2. 엔드포인트 노출     
엔드포인트를 활성화하고 추가로 HTTP를 통해서 웹에 노출할지, 아니면 JMX를 통해서 노출할지 두    
위치에 모두 노출할지 노출 위치를 지정해주어야 한다.    
#### 엔드포인트 목록
- beans : 스프링 컨테이너에 등록된 스프링 빈을 보여준다.
- conditions : condition 을 통해서 빈을 등록할 때 평가 조건과 일치하거나 일치하지 않는 이유를
표시한다.
- configprops : @ConfigurationProperties 를 보여준다.
- env : Environment 정보를 보여준다.
- health : 애플리케이션 헬스 정보를 보여준다.
- httpexchanges : HTTP 호출 응답 정보를 보여준다. HttpExchangeRepository 를 구현한 빈을 별도로
등록해야 한다.
- info : 애플리케이션 정보를 보여준다.
- loggers : 애플리케이션 로거 설정을 보여주고 변경도 할 수 있다.
- metrics : 애플리케이션의 메트릭 정보를 보여준다.
- mappings : @RequestMapping 정보를 보여준다.
- threaddump : 쓰레드 덤프를 실행해서 보여준다.
- shutdown : 애플리케이션을 종료한다. 이 기능은 기본으로 비활성화 되어 있다.
 #### health
- 헬스 정보를 사용하면 애플리케이션에 문제가 발생했을 때 문제를 빠르게 인지
 #### info 엔드포인트
- info 엔드포인트는 애플리케이션의 기본 정보를 노출
- java : 자바 런타임 정보
- os : OS 정보
- env : Environment 에서 info. 로 시작하는 정보
- build : 빌드 정보, META-INF/build-info.properties 파일이 필요하다.
- git : git 정보, git.properties 파일이 필요하다.
 #### 로거
- loggers 엔드포인트를 사용하면 로깅과 관련된 정보를 확인하고, 또 실시간으로 변경할 수도 있다. 
- loggers 엔드포인트를 사용하면 애플리케이션을 다시 시작하지 않고, 실시간으로 로그 레벨을 변경할 수
있다.
 #### HTTP 요청 응답 기록
 - HTTP 요청과 응답의 과거 기록을 확인하고 싶다면 httpexchanges 엔드포인트를 사용하면 된다.
HttpExchangeRepository 인터페이스의 구현체를 빈으로 등록하면 httpexchanges 엔드포인트를
사용할 수 있다.
 - (주의! 해당 빈을 등록하지 않으면 httpexchanges 엔드포인트가 활성화 되지 않는다)
 - 스프링 부트는 기본으로 InMemoryHttpExchangeRepository 구현체를 제공한다.
### 액츄에이터와 보안
#### 보안 주의
-액츄에이터가 제공하는 기능들은 우리 애플리케이션의 내부 정보를 너무 많이 노출한다. 그래서 외부   
인터넷 망이 공개된 곳에 액츄에이터의 엔드포인트를 공개하는 것은 보안상 좋은 방안이 아니다.    
액츄에이터의 엔드포인트들은 외부 인터넷에서 접근이 불가능하게 막고, 내부에서만 접근 가능한 내부망을   
사용하는 것이 안전하다.   
#### 1. 액츄에이터를 다른 포트에서 실행
예를 들어서 외부 인터넷 망을 통해서 8080 포트에만 접근할 수 있고, 다른 포트는 내부망에서만 접근할 수
있다면 액츄에이터에 다른 포트를 설정하면 된다.
액츄에이터의 기능을 애플리케이션 서버와는 다른 포트에서 실행하려면 다음과 같이 설정하면 된다. 이
경우 기존 8080 포트에서는 액츄에이터를 접근할 수 없다.
#### 2. 액츄에이터 URL 경로에 인증 설정
포트를 분리하는 것이 어렵고 어쩔 수 없이 외부 인터넷 망을 통해서 접근해야 한다면 /actuator 경로에
서블릿 필터, 스프링 인터셉터 또는 스프링 시큐티리를 통해서 인증된 사용자만 접근 가능하도록 추가
개발이 필요하다.