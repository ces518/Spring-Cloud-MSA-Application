# Docker - RabbitMQ

## RabbitMQ 컨테이너 생성

```shell
docker run -d --name rabbitmq --network ecommerce-network \
	-p 15672:15672 -p 5672:5672 -p 15671:15671 -p 5671:5671 -p 4369:4369 \
	-e RABBITMQ_DEFAULT_USER=guest \
	-e RABBITMQ_DEFAULT_PASS=guest rabbitmq:management
```
- 이전에 생성한 network (ecommerce-network) 를 사용하도록 지정한다.
    - 지정하지 않을 경우 docker 가 사용하는 기본 bridge network 를 사용한다


`RabbitMQ 컨테이너 생성 후 확인`

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
- Containers 항목을 보면 Rabbit MQ 컨테이너 등록된 것을 확인할 수 있다.
