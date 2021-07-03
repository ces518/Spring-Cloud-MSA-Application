# Kafka Sink Connector

## Sink Connect 테스트
- Sink Connect 의 역할은, Source Connect 가 발행한 메세지를 소비하는 역할

```shell
curl -X "POST" "http://localhost:8083/connectors" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "name": "my-sink-connect",
  "config": {
    "topics": "my_topic_users",
    "connection.password": "root!",
    "table.name.format": "my_users", // 연결할 테이블 이름 
    "delete.enabled": "false",
    "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "auto.create": "true",  // 테이블이 존재하지 않으면 자동으로 table 을 생성 한다.
    "tasks.max": "1",
    "auto.evolve": "true",
    "connection.user": "root",
    "connection.url": "jdbc:mysql://localhost:3306/mydb"
  }
}'
```

```shell
MariaDB [mydb]> show tables;
+----------------+
| Tables_in_mydb |
+----------------+
| my_users       |
| users          |
+----------------+

MariaDB [mydb]> select * from my_users;
+----+---------+----------+------------+-------------------------+
| id | user_id | pwd      | name       | created_at              |
+----+---------+----------+------------+-------------------------+
|  1 | user1   | test1111 | User name  | 2021-07-03 19:21:09.000 |
|  2 | admin1  | test1111 | Admin name | 2021-07-03 19:30:04.000 |
+----+---------+----------+------------+-------------------------+
```
- Sink Connector 가 정상 생성되면 지정된 테이블과 데이터가 저장된걸 확인할 수 있다.

## 신규 데이터 등록

```shell
MariaDB [mydb]> insert into users(user_id, pwd, name) values('admin123', 'test1111', 'Admin name123');
Query OK, 1 row affected (0.002 sec)

MariaDB [mydb]> commit;
Query OK, 0 rows affected (0.000 sec)

MariaDB [mydb]> select * from my_users;
+----+----------+----------+---------------+-------------------------+
| id | user_id  | pwd      | name          | created_at              |
+----+----------+----------+---------------+-------------------------+
|  1 | user1    | test1111 | User name     | 2021-07-03 19:21:09.000 |
|  2 | admin1   | test1111 | Admin name    | 2021-07-03 19:30:04.000 |
|  3 | admin123 | test1111 | Admin name123 | 2021-07-03 20:44:15.000 |
+----+----------+----------+---------------+-------------------------+
3 rows in set (0.001 sec)
```
- Sink Connect Table 에도 정상적으로 데이터가 등록된걸 확인할 수 있다.