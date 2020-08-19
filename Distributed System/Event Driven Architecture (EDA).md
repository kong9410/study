## Event Driven Architecture (EDA)

분산된 시스템 간에 이벤트를 생성, 발행하고 발행된 이벤트를 필요로하는 수신자에게 전송됨

이벤트를 수신한 수신자가 이벤트를 처리하는 형태의 시스템 아키텍쳐

주로 Message Broker(Kafka, Rabbit MQ, Redis)와 결합하여, Message Driven 시스템으로 구성됨



### 구성요소

- Event generator : 시스템 내, 외부의 상태변화를 감지하여 표준화된 형식의 이벤트 생성
- Event channel : 이벤트를 필요로 하는 시스템까지 전송
- Event processing engine : 수신한 이벤트를 식별, 적절한 처리를 함. 때에 따라 결과로 또다른 이벤트를 발생시킬 수 있음.



### Event Processing Style

수신한 이벤트 처리 방법

- Simple event processing : 각 이벤트가 직접적으로 수행해야할 action과 매핑되어 처리됨. 실시간 정보의 흐름을 처리할 때 사용, 이벤트 처리 시간과 비용 손실이 적음
- Event Stream processing : 이벤트 중요도에 따라 필터링하여 걸러진 이벤트만을 수신자에게 전송. 실시간 정보의 흐름을 처리할 때 사용
- Complex event processing : 일상적인 이벤트 패턴을 감지하여 더 복잡한 이벤트 발생을 추론



## Event Driven Architecture 장단점

#### 장점

- Decoupling : 시스템 간의 느슨한 결합이 가능 하므로 분산 시스템, Microservice 환경에서 시스템 간 의존성을 배제할 수 있다.
- 다른 시스템의 정보를 알 필요가 없다.
- Micro Service 단위로 시스템을 분리하기 쉽기 때문에 확장성, 탄력성을 고려하기 쉽다.

#### 단점

- Broker Dependency : Event를 전송하기 위한 Message Broker에 대한 의존성이 커지기 때문에 Message Broker 장애 상황 시, 전체 장애로 이어질 수 있음
- Transaction 단위가 격리되기 때문에 서비스 장애 발생시 retry/rollback을 고려해야함
- 시스템 전체 Flow를 파악하기 어렵다.
- 디버깅이 어렵다.

