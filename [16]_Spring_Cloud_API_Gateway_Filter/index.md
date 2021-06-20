# Spring Cloud API Gateway Filter

## Spring Cloud API Gateway

- Predicate (조건에 따라 분기 역할)
- Pre Filter (사전 필터)
- Post Filter (사후 필터)

> 필터를 적용하는 방식은 Java Code / Yaml 작성 모두 지원한다.

## Java Code 로 Filter 적용

```java

@Configuration
public class FilterConfig {

	@Bean
	public RouteLocator gatewayRoutes(RouteLocatorBuilder builder) {
		return builder.routes()
				.route(r -> r.path("/first-service/**")
						.filters(f ->
								f.addRequestHeader("first-request", "first-request-header")
										.addResponseHeader("first-response", "first-response-header")
						)
						.uri("http://localhost:8081/"))
				.route(r -> r.path("/second-service/**")
						.filters(f ->
								f.addRequestHeader("second-request", "second-request-header")
										.addResponseHeader("second-response", "second-response-header")
						)
						.uri("http://localhost:8082/"))
				.build();
	}
}
```

## Yaml 로 Filter 적용

```yaml
spring:
  application:
    name: apigateway-service
  cloud:
    gateway:
      routes:
        - id: first-service
          uri: http://localhost:8081/
          predicates:
            - Path=/first-service/** # http://localhost:8081/first-serivce/** 형태로 그대로 전달됨을 주의..
          filters:
            - AddRequestHeader=first-request, first-request-header
            - AddResponseHeader=first-response, first-response-reader
        - id: second-service
          uri: http://localhost:8082/
          predicates:
            - Path=/second-service/**
          filters:
            - AddRequestHeader=second-request, second-request-header
            - AddResponseHeader=second-response, second-response-reader
```