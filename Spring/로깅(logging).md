# logging

### Log4j 특징

- 속도 최적화
- 멀티 스레드 환경에서 적합
- 계층적 로그 설정과 처리를 지원
- 출력을 파일, 콘솔, outputstream, wrtier, tcp를 사용하는 원격서버 등에 보낼 수 있다.
- 계층적 6가지 로그 메시지 레벨을 제공 (TRACE, DEBUG, INFO, WARN, ERROR, FATAL)

#### 장점

- 문제 파악에 용이
- 빠른 디버깅 가능
- 로그 파악이 쉬움
- 로그 이력을 파일, DB로 남길 수 있음
- 효율적인 디버깅 가능

#### 단점

- 로그에 대한 디바이스 입출력으로 인해 런타임 오버헤드가 발생
- 로깅을 위한 추가 코드로 인해 전체 코드 사이즈 증가
- 심하게 생성되는 로그는 혼란을 야기
- 심하게 생성되는 로그는 성능 문제를 야기
- 개발 중간 로깅 코드 추가가 어려움



### SLF4J

logger를 변경해야 하는 작업이 있을 수 있을 때, logger를 통일하고 싶다면 slf4j를 쓰면 편하다

SLF4J는 로깅 구현체가 아닌 Logging Facade로 로깅에 대한 추상 레이어를 제공한다. (interface 덩어리)



### Logback

- log4j를 토대로 새롭게 만든 Logging 라이브러리

- slf4j를 통해 다른 logging framework를 쓰더라도 logback으로 통합 가능
- 주요 설정 요소
  - Logger : 로깅을 수행하는 요소 : Level 속성을 통해 출력할 로그 레벨 조절 가능
  - Appender : 로그 메시지가 출력될 대상을 결정하는 요소
  - Encoder : Appender에 포함되어 사용자가 지정한 형식으로 표현 될 로그메시지를 변환하는 역할을 담당하는 요소
  - XML을 이용한 설정 : logback.xml을 작성 후 클래스 패스에 위치시킴
  - Groovy를 이용한 설정 : logback.groovy로 설정 파일 작성 후 파일을 클래스패스에 위치

#### 장점

- log4j보다 약 10배 정도 빠르게 수행되도록 내부가 변경되었음
- 메모리 효율이 좋음
- 문서화가 잘되어 있음
- 설정 파일을 변경했을 때, 서버 재기동 없이 변경 내용이 자동으로 갱신
- 서버 중지 없이 I/O Failure에 대한 복구를 지원
- RollingFileAppender를 사용할 경우 자동적으로 오래된 로그를 지우고 Rolling 백업을 처리

#### 로깅 레벨

- TRACE
- DEBUG
- INFO
- WARN
- ERROR


#### 원문

https://goddaehee.tistory.com/45



