# 아파치 카프카
- Scalar 로 개발된 메세지 브로커
- 링크드인에서 개발, 2011년 오픈소스화
- Confluent 회사 창립
- 실시간 데이터 피드를 관리하기 위해 통일된 높은 처리량, 낮은 지연 시간을 지닌 플랫폼 제ㅗㅇ
- Apple, Netflix, Kakao. EBay, Paypal, Uber 등 에서 사용중

## 카프카 사용 이전
- End to End 연결 방식의 아키텍쳐
- 데이터 연동의 복잡성이 증가 (HW, OS, 장애 등)
- 서로 다른 데이터 Pipeline 연결 구조
- 확장이 어려운 구조

## 카프카 데이터 처리 흐름
- 프로듀서 / 컨슈머 분리
- 메세지를 여러 컨슈머에게 허용
- 높은 처리량을 위한 메세지 최적화
- 스케일 아웃
- 에코 시스템
- 데이터 스트리밍 / RDB 의 SQL 과 같은 기능을 제공

## 카프카 브로커
- 실행된 카프카 서버
- 3대 이상의 브로커를 클러스터로 구성한다.
- Zookeeper 를 연동
    - 메타 데이터 (Broker ID, Controller ID), Controller 정보 저장
- N 개의 브로커 중 1대는 Controller 역할 (리더) 을 수행한다.
    - Controller 는 각 브로커에게 담당 파티션 할당을 수행
    - 브로커 모니터링

## Docker-Compose 로 설치

```shell
// Kafka-Docker repo clone
git clone https://github.com/wurstmeister/kafka-docker

// docker-compose.yml 파일 수정
vim docker-compose-single-broker.yml

version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
  kafka:
    build: .
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_CREATE_TOPICS: "test:1:1"
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      
// docker-compose 실행
docker-compose -f docker-compose-single-broker.yml up -d
```

## Kafka Java Client
- https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients

## Zookeeper 및 Kafka 서버 기동
$KAFKA_HOME/bin/zookeeper-server-start.sh  $KAFKA_HOME/config/zookeeper.properties

$KAFKA_HOME/bin/kafka-server-start.sh  $KAFKA_HOME/config/server.properties

## Topic 생성
$KAFKA_HOME/bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092 \

--partitions 1

## Topic 목록 확인
$KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

## Topic 정보 확인
$KAFKA_HOME/bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092

## 메시지 생산
$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic quickstart-events

## 메시지 소비
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events \

--from-beginning // 처음부터 메세지를 읽는 옵션

