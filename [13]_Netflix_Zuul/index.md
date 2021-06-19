# Netflix Zuul

## Zuul 프로젝트 생성
- Spring Boot 2.3.x 버전까지만 사용 가능함
- Service Discovery 에 등록하지 않고 Zuul 을 통해 사용하도록 세팅 

`Zuul Service`

```yaml
sever:
  port: 8000

spring:
  application:
    name: my-zuul-service

zuul:
  routes:
    first-service:
      path: /first-service/**
      url: http://localhost:8081
    second-service:
      path: /second-service/**
      url: http://localhost:8082
```

`FirstService`

```yaml
server:
  port: 8081

spring:
  application:
    name: my-first-service

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

`SecondService`

```yaml
server:
  port: 8082

spring:
  application:
    name: my-first-service

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

- http://localhost:8080/first-service/welcome
- http://localhost:8080/second-service/welcome
- 위 URI 로 요청시 Zuul 을 통해 라우팅된다..

## 참고
- Netflix Zuul 과 Spring Cloud Gateway 는 차이가 있음
    - 아키텍쳐 적인 부분의 변화가 많음
- https://happycloud-lee.tistory.com/218