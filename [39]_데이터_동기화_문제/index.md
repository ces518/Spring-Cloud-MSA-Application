# 데이터 동기화 문제

## Multi Instance
- Order Service 2 개 기동중
    - H2 데이터베이스를 사용중..
    - Users 요청 분산 처리
    - Orders 데이터도 분산 저장중 -> 데이터 동기화 문제가 발생함

`해결책`
- 하나의 데이터베이스를 사용한다.
- 데이터베이스 간의 동기화를 시킨다.
    - MessageQueueServer 사용
    
> **KafkaConnector** + DB 를 사용한다. \n
> 각 마이크로서비스가 Kafka 로 메시지를 보내고 Kafka Connector 를 이용해 DB 에 동기화를 한다.


