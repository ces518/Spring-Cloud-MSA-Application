# Docker - MariaDB

## MariaDB 컨테이너 생성

`DockerFile`

```dockerfile
FROM mariadb
ENV MYSQL_ROOT_PASSWORD root!
ENV MYSQL_DATABASE mydb
COPY ./db /var/lib/mysql
EXPOSE 3306
ENTRYPOINT ["mysqld", "--user=root"]
```
- 기존에 로컬에 설치된 mysql 폴더를 ./db 폴더로 복사해야 한다.

`Docker Image Build`

```shell
docker build -t ces518/maraidb:1.0 .

docker images
REPOSITORY                                  TAG                IMAGE ID       CREATED          SIZE
ces518/mariadb                              1.0                bd9db2fc6613   58 seconds ago   546MB
```

`MariaDB 실행`

```shell
docker run -d -p 3306:3306  --network ecommerce-network --name mariadb ces518/mariadb:1.0
```
