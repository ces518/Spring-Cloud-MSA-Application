# User Service Load Balancer

## 여러 Instance 가동

```shell
$ mvn spring-boot:run -Dspring-boot.run.jvmArguments='-Dserver.port=[포트번호]'

$ java -jar -Dserver.port=9004 ./target/user-service-0.0.1-SNAPSHOT.jar
```

> 매번 포트번호를 지정해서 하는방식은 불편하다... (오토 스케일링 작업시 불편함)

## Random Port 를 지정하기

```yaml
server:
  port: 9001 # 포트를 0번으로 지정하면 가용가능한 범위 내에서 랜덤한 포트로 실행됨

spring:
  application:
    name: user-service
```

## Random Port 를 사용할 경우 문제
- ${PORT}:${Spring.application.name}:${yml의 port} 로 인스턴스를 표기한다.
- 위 표기방식 때문에 Random Port 로 지정할 경우 포트번호 0번에 대해서 인스턴스 1개의 정보만 노출된다.
- 이를 해결하기 위해 eureka.instance.instance-id 프로퍼티 설정이 필요함
```yaml
# Eureka Server 의 기본 표기법 = ${PORT}:${Spring.application.name}:${yml의 port} 로 인스턴스를 표기한다
# Random Port 로 기동할 경우 0 port 에 대해서만 표기가 된다.. => 이는 표기법에 대한 문제
# instance-id 에 대한 설정이 필요함
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8761/eureka # Eureka 서버 정보
  instance:
    instance-id: ${spring.cloud.client.hostname}:${spring.application.instance_id:${random.value}}
```
