# Kafka Source Connector

## Source Connector 테스트

`Connector 등록`

```shell
curl -X "POST" "http://localhost:8083/connectors" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "name": "my-source-connect", // 커넥터 명
  "config": {
    "incrementing.column.name": "id",
    "connection.password": "root!",
    "topic.prefix": "my_topic_",
    "mode": "incrementing",
    "table.whitelist": "users",
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector", // jdbc source connect 사용
    "tasks.max": "1",
    "connection.user": "root",
    "connection.url": "jdbc:mysql://localhost:3306/mydb"
  }
}'
```

`Connector 상태 확인`

```shell
localhost:8083/connectors/my-source-connect/status

{"name":"my-source-connect","connector":{"state":"RUNNING","worker_id":"127.0.0.1:8083"},"tasks":[{"id":0,"state":"RUNNING","worker_id":"127.0.0.1:8083"}],"type":"source"}
```

`MaraiDB Data Insert`

```shell
MariaDB [mydb]> insert into users(user_id, pwd, name) values('user1', 'test1111', 'User name');
```

`Kafka Topics`

```shell
kafka-topics.sh --bootstrap-server localhost:9092 --list
__consumer_offsets
connect-configs
connect-offsets
connect-status
my_topic_users
```

- data 를 추가하고 나면 새로운 토픽이 추가된걸 확인할 수 있다.

`Consume Topics`

```shell
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my_topic_users --from-beginning
{"schema":{"type":"struct","fields":[{"type":"int32","optional":false,"field":"id"},{"type":"string","optional":true,"field":"user_id"},{"type":"string","optional":true,"field":"pwd"},{"type":"string","optional":true,"field":"name"},{"type":"int64","optional":true,"name":"org.apache.kafka.connect.data.Timestamp","version":1,"field":"created_at"}],"optional":false,"name":"users"}
,"payload":{"id":1,"user_id":"user1","pwd":"test1111","name":"User name","created_at":1625340069000}
}
{"schema":{"type":"struct","fields":[{"type":"int32","optional":false,"field":"id"},{"type":"string","optional":true,"field":"user_id"},{"type":"string","optional":true,"field":"pwd"},{"type":"string","optional":true,"field":"name"},{"type":"int64","optional":true,"name":"org.apache.kafka.connect.data.Timestamp","version":1,"field":"created_at"}],"optional":false,"name":"users"}
,"payload":{"id":2,"user_id":"admin1","pwd":"test1111","name":"Admin name","created_at":1625340604000}
}
```
- data 가 추가되면 위와 같은 topic 에 메세지가 producing 된다.
