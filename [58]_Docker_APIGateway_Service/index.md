# Docker - API Gateway Service

## API Gateway Service 컨테이너 생성

`DockerFile`

```dockerfile
FROM openjdk:17-ea-11-jdk-slim
VOLUME /tmp
COPY target/APIGatewayService-1.0.jar ApiGatewayService.jar
ENTRYPOINT ["java", "-jar", "ApiGatewayService.jar"]
```

`Docker Image Build`

```shell
docker build -t ces518/apigateway-service:1.0 .

docker images
REPOSITORY                                  TAG                IMAGE ID       CREATED             SIZE
ces518/apigateway-service                   1.0                6bd8d41624f5   11 seconds ago      446MB
```

`API Gateway Service 실행`

```shell
docker run -d -p 8000:8000 --network ecommerce-network \
 -e "spring.cloud.config.uri=http://config-service:8888" \
 -e "spring.rabbitmq.host=rabbitmq" \
 -e "eureka.client.serviceUrl.defaultZone=http://discovery-service:8761/eureka/" \
 --name apigateway-service \
 ces518/apigateway-service:1.0
```

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
            "2bd76f26b0acd9dc2903b6224e5d440145f122c46692a1e8b3fd95578e90c996": {
                "Name": "rabbitmq",
                "EndpointID": "a76ee49d70448964b178345d9d1e698df1a27dd970dbb13e7edc62b2e35d35f2",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "7cc49bf5d101ae370121f9da4796eead9a3451d0faba639b480e1df06cec2d52": {
                "Name": "config-server",
                "EndpointID": "fdd17b116f99f03bbdcf0406eed8eac4b02a846d6ceaa27a661c6692a80c5ee6",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "ad6f5b4fb6f463e31776e8985eb9b451203ab8b8ea944331b98986a08e05826c": {
                "Name": "discovery-service",
                "EndpointID": "9ed9670feddc621fae93acf3dbecf19c2a7e3cf12f7a33018dc09df929e37425",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "ba34e48ee7c2e04ac28b2ef628ba44d8ed2af55ea8a96120fd9af96b4b5ab715": {
                "Name": "apigateway-service",
                "EndpointID": "9bd15d4682374147f0bb6180709cb740f460375ae351bb0bc505b2125709c3ad",
                "MacAddress": "02:42:ac:12:00:05",
                "IPv4Address": "172.18.0.5/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```