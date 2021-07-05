# CircuitBreaker Resilience4j

## MSA 통신시 연쇄 오류
- UserService -> OrderService -> CatalogService
                             [ 에러 발생 ]

> 위와 같은 문제 발생시, UserService 의 문제가 아님에도 500 에러가 발생할 수 있다. \n
> OrderService 혹은 CatalogService 에 문제가 발생하더라도 UserService 에 문제가 발생하면 안된다.

## CircuitBreaker ?
- 장애가 발생하는 서비스를 반복적인호출되지 못하게 차단해주는 역할
- 전기 회로 차단기에서 착안됨
- 특정 서비스가 정상 동작하지 않을 경우 다른 기능으로 대체 수행한다.
    - 장애 회피

- Spring Cloud 2020.0.0 이전 버전에서는 spring-cloud-netflix-hystrix 를 추가해 주어야한다.
- netflix-hystrix 는 EOD, 2.3.x 이후 버전부터 라이브러리를 제공하지 않는다.
- Functional Programming 을 지원하지 않음

## Resilience4j
- circuitbreaker
- ratelimiter
- bulkhead
- ety
- timelimiter
- cache
- Functional Programming 지원

## 설정

`의존성 추가`

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

`설정`

```java
@Configuration
public class Resilience4JConfig {

	@Bean
	public Customizer<Resilience4JCircuitBreakerFactory> globalCustomConfiguration() {
		CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
			.failureRateThreshold(4) // Circuit Breaker 를 Open 할지 결정하는 failure Rate (default: 50)
			.waitDurationInOpenState(Duration.ofMillis(1000)) // Circuit Breaker Open 상태를 유지하는 시간, 이 기간 이후 half-open 상태 (default: 60)
			.slidingWindowType(CircuitBreakerConfig.SlidingWindowType.COUNT_BASED) // Circuit 을 닫을때 결과 기록시 Count 기반 / Time 기반
			.slidingWindowSize(2) // Circuit 이 닫힐때 호출 결과를 기록시 사용되는 Sliding Window 의 크기 (default: 100)
			.build();

		TimeLimiterConfig timeLimiterConfig = TimeLimiterConfig.custom()
			.timeoutDuration(Duration.ofSeconds(4)) // 해당 서비스가 4초내에 응답이 오지 않을경우 Circuit open 짖어하는 time limit
			.build();

		return factory -> factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
			.circuitBreakerConfig(circuitBreakerConfig)
			.timeLimiterConfig(timeLimiterConfig)
			.build()
		);
	}
}
```
- failureRateThreshold : Circuit 을 Open 할지 결정하는 failure Rate (default: 50)
- waitDurationInOpenState : Circuit Open 상태를 유지하는 기간, 이 기간 이후 half-open 상태가 된다. (default: 60)
    - half-open 상태가 될 경우 해당 서비스가 정상화 되었는지 확인하고, 정상화 되었다면 close 된다.
- slidingWindowType : Circuit 을 Close 할 때 결과를 기록하는데, 이를 Count 기반 / Time 기반으로 할지 결정
- slidingWindowSize : Circuit 이 닫힐 때 호출 결과를 기록하는데 기록시 사용되는 Sliding Window 의 크기 (default: 100)
- timeoutDuration : 해당 서비스가 지정한 timeout 내에 응답이 오지 않을 경우 Circuit 을 Open 한다

```java

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class UserService implements UserDetailsService {
	private final UserRepository repository;
	private final PasswordEncoder passwordEncoder;
	private final ModelMapper mapper;

	private final RestTemplate restTemplate;
	private final Environment env;

	private final OrderServiceClient orderServiceClient;
	private final CircuitBreakerFactory circuitBreakerFactory;

	public UserDto getUserById(String userId) {
		User user = repository.findByUserId(userId)
			.orElseThrow(RuntimeException::new);
		UserDto userDto = mapper.map(user, UserDto.class);

		/*
		RestTemplate
		String orderUrl = String.format(env.getProperty("order_service.url"), userId);
		ResponseEntity<List<ResponseOrder>> ordersResponse = restTemplate.exchange(
			orderUrl,
			HttpMethod.GET,
			null,
			new ParameterizedTypeReference<List<ResponseOrder>>() {} // Super Type Token
		);
		List<ResponseOrder> responseOrders = ordersResponse.getBody();
		*/


		// FeignClient
//		List<ResponseOrder> responseOrders = orderServiceClient.getOrders(userId);

		// CircuitBreaker
		CircuitBreaker circuitbreaker = circuitBreakerFactory.create("circuitbreaker");
		List<ResponseOrder> responseOrders = circuitbreaker.run(() -> orderServiceClient.getOrders(userId),
			throwable -> new ArrayList<>());

		userDto.setOrders(responseOrders);
		return userDto;
	}
}
```