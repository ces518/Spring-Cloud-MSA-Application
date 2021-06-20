# Spring Cloud API Gateway

## Zuul vs SCG
- Zuul 과 유사하지만, SCG 와의 목적이 다름
    - SCG 는 비동기 처리가 가능하다.
- Zuul 1.x 일 경우 동기식으로 동작 했다. (Servlet 기반)
- Zuul 2.x 인 경우 비동기식으로 지원한다고 했지만, Spring lib 와 호환문제 때문에 스프링 진영에서 별도로 만든 프로젝트이다.

## API Gateway 설정

```yaml
server:
  port: 8000

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:8761/eureka
spring:
  application:
    name: apigateway-service
  cloud:
    gateway:
      routes:
        - id: first-service
          uri: http://localhost:8081/
          predicates:
            - Path=/first-serivce/** # http://localhost:8081/first-serivce/** 형태로 그대로 전달됨을 주의..
        - id: second-service
          uri: http://localhost:8082/
          predicates:
            - Path=/second-serivce/**
```
- Zuul 과 설정은 유사하지만 프로퍼티가 일부 변경되었음에 유의
- API Gateway 로 부터 전달받은 Path가 그대로 forward 된다는 점을 주의해야한다.
