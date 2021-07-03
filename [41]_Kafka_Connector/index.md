# Kafka Connector

- Kafka Connect 를 통해 Data Import/Export 가능
- 코드 없이 Configuration 만으로 데이터 이동
- Standalone Mode, Distribution (분산) Mode 지원
    - RESTful APi
    - Stream / Batch 형태로 전송
    - Custom Connector 를 통해 다양한 플러그인 제공 (File, S3, Hive, MySQL, ETC ...)

`예시`

Source System (Hive) -> Kafka Connect Source -> Kafka Cluster -> Kafka Connect Sink -> Target System (S3 ...)

## Docker-Compose : Mariadb

`docker-compose.yml`

```yaml
version: "3"

services:
  db:
    image: mariadb:10
    ports:
      - 3306:3306
    volumes:
      - ./db/conf.d:/etc/mysql/conf.d
      - ./db/data:/var/lib/mysql
      - ./db/initdb.d:/docker-entrypoint-initdb.d
    env_file: ./db/.env
    environment:
      TZ: Asia/Seoul
    networks:
      - backend
    restart: always

networks:
  backend:
```

`.env`

```shell
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server           = utf8mb4
collation-server               = utf8mb4_unicode_ci
```

```shell
docker-compose -f docker-compose.yml up -d
```

- https://github.com/jolbon/jolbon-api

## H2-Console : Mariadb
- Order-Service 에 Mariadb 를 연동한다.
- 기존 H2-Console 은 H2 뿐만이 아니라 다양한 데이베이스를 지원함.
- 단 커넥션을 맺기위해 해당 JDBC 드라이버가 필요하다.

```xml
<dependency>
    <groupId>org.mariadb.jdbc</groupId>
    <artifactId>mariadb-java-client</artifactId>
    <version>2.7.2</version>
</dependency>
```

## Kafka Connect 설치

```shell
curl -O http://packages.confluent.io/archive/6.1/confluent-community-6.1.0.tar.gz
tar xvf confluent-community-6.1.0.tar.gz

// 실행
./bin/connect-distributed ./etc/kafka/connect-distributed.properties

// Kafka Connect 실행후 Kafka 의 Topic 들 조회
bash-5.1# kafka-topics.sh --bootstrap-server localhost:9092 --list
__consumer_offsets
connect-configs
connect-offsets
connect-status
```
- Kafka Connect 실행후 Kafka Connect 와 관련된 topic 들이 추가되어 있다.

```shell
/kafka-connec-java/confluentinc-kafka-connect-jdbc-10.2.0/lib/kafka-connect-jdbc-10.2.0.jar
```
- connect-distributed.properties 설정 파일에 lib 경로를 추가해주어야한다.

`Kafka Connect`
- /kafka-connect/confluent-6.1.0/share/java/kafka 에 maria-db-connector 라이브러리를 추가해 주어야 Jdbc Connection 을 맺을 수 있다.