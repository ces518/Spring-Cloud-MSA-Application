# FeignClient
- REST Call 을 추상화한 Spring Cloud Netflix 라이브러리
- 호출하려는 HTTP Endpoint 에 대한 Interface 를 생성
    - @FeignClient 선언
- @LoadBalanced 를 지원

> 훨씬 더 직관적이고 사용하기 편하다.

## 설정
- Spring Cloud Starter OpenFeign 
- @EnableFeignClients

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableDiscoveryClient // DiscoveryClient 를 구현한것이 EurekaClient ...
@EnableFeignClients
public class UserServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(UserServiceApplication.class, args);
	}

	@Bean
	@LoadBalanced
	public RestTemplate restTemplate() {
		return new RestTemplate();
	}
}
```

## 예외 처리
- FeignClient 의 예외처리는, ErrorDecoder / FallbackFactory 두가지 방식을 지원한다.
- ErrorDecoder 는 FeignClient 의 Error Handling 에 사용되고
- FallbackFactory 는 Hystrix 와 함께 사용된다.
    - circuit 이 open 되거나 실행 중 에러가 발생한 경우 실행된다.

`ErrorDecoder`

```java
@Component
public class FeignErrorDecoder implements ErrorDecoder {

	@Override
	public Exception decode(String methodKey, Response response) {
		switch (response.status()) {
			case 400:
				break;
			case 404:
				if (methodKey.contains("getOrders")) {
					return new ResponseStatusException(
						HttpStatus.valueOf(response.status()),
						"User's order is empty"
					);
				}
				break;
			case 500:
				break;
			default:
				return new Exception(response.reason());
		}
		return null;
	}
}
```