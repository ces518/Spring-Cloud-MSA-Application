# Spring Cloud API Gateway Custom Filter

## Spring Cloud API Gateway Custom Filter
- CustomFilter 는 AbstractGatewayFilterFactory 클래스를 를 구현 해야 한다

```java
@Slf4j
@Component
public class CustomFilter extends AbstractGatewayFilterFactory<CustomFilter.Config> {

	public CustomFilter() {
		super(Config.class);
	}

	@Override
	public GatewayFilter apply(Config config) {
		// Custom Pre Filter
		return (exchange, chain) -> {
			ServerHttpRequest request = exchange.getRequest();
			ServerHttpResponse response = exchange.getResponse();
			log.info("Custom Pre Filter: request id -> {} ", request.getId());

			// Custom Post Filter
			// Mono -> 비동기식 지원
			return chain.filter(exchange).then(Mono.fromRunnable(() -> {
				log.info("Custom POST Filter : response code -> {}", response.getStatusCode());
			}));
		};
	}

	static class Config {

	}
}
```
- Lambda 로 구현했지만 실제론 OrderedGatewayFilter 와 같은 구현체를 반환해야 한다.

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
#            - AddRequestHeader=first-request, first-request-header
#            - AddResponseHeader=first-response, first-response-reader
            - CustomFilter
        - id: second-service
          uri: http://localhost:8082/
          predicates:
            - Path=/second-service/**
          filters:
#            - AddRequestHeader=second-request, second-request-header
#            - AddResponseHeader=second-response, second-response-reader
            - CustomFilter
```
- 별도로 구현한 CustomFilter 를 yaml filters 프로퍼티에 매핑하면 해당 필터가 적용된다.