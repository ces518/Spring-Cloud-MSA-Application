# 컨테이너 실행 과 생성

## MySQL 실행

```shell
docker run -d -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true --name mysql mysql:5.7
```
- MySQL 은 root password 를 반드시 지정해주어야 실행된다.
- root password 를 지정하지 않도록 MYSQL_ALLOW_EMPTY_PASSWORD=true 환경변수를 지정한다.

```shell
// mysql bash 로 진입
docker exec -it mysql bash

// mysql 접속, root password 를 지정하지 않았기때문에 바로 접속됨
mysql root

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.33 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```