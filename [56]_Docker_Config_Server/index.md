# Docker - Config Server

## Config Server 컨테이너 생성
- EXPORT 된 JAR 파일과, ENCRYPTION 파일이 컨테이너에 포함되어 있어야 한다.

`ENCRYPTION 파일을 project root 로 복사`

```shell
cp apiEncryptionKey.jks /Users/kakaocommerce/work/ECommerceService/ConfigServer
```


`DockerFile`

```dockerfile
FROM openjdk:17-ea-11-jdk-slim
VOLUME /tmp
COPY apiEncryptionKey.jks apiEncryptionKey.jks
COPY target/ConfigServer-1.0.jar ConfigServer.jar
ENTRYPOINT ["java", "-jar", "ConfigServer.jar"]
```

`Docker Image Build`

```shell
// 이미지 빌드
docker build -t ces518/config-server:1.0 .

// 이미지 생성 확인
docker images
REPOSITORY                                  TAG                IMAGE ID       CREATED          SIZE
ces518/config-server                        1.0                62d83f2a2b7a   29 seconds ago   437MB
```

`ConfigServer 실행`

```shell
docker run -d -p 8888:8888 --network ecommerce-network \
    -e "spring.rabbitmq.host=rabbitmq" \
    -e "spring.profiles.active=default" \
    --name config-server ces518/config-server:1.0
```
- rabbitmq host 에 대한 정보를 환경변수로 제공한다.
- ecommerce-network 에 존재하는 rabbitmq host 에 접근하도록 변경

`Docker Network 에 포함 되었는지 확인`

```shell
 docker network inspect ecommerce-network
[
    {
        "Name": "ecommerce-network",
        "Id": "54ee77113bc8feb13ba801527ce8e189b537ca55f7fef0093c9cfd6dee1617d0",
        "Created": "2021-07-13T13:10:18.942476743Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "1a65212089eda80e7e0d3aef42b35d523bcfb06a6a99c35dde1adc26fab7cf28": {
                "Name": "config-server",
                "EndpointID": "d11fea47cc76a22832084710091e5422639ab91f1a32feb2d1e97c2c29157c02",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "2bd76f26b0acd9dc2903b6224e5d440145f122c46692a1e8b3fd95578e90c996": {
                "Name": "rabbitmq",
                "EndpointID": "d8306028cfa047f21088f679988bfcc760ad64fea32ea334e014d317f355c09f",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```