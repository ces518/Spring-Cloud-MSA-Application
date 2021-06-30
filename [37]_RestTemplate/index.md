# RestTemplate

## RestTemplate 이란 ?
- org.springframework.http.client 패키지
- Spring 3.0 버전부터 지원
- **Thread-Safe**
- JdbcTemplate 과 같이 반복적인 코드를 사용하기 편하게 추상화 해둔 클래스
- 내부적으로 HTTP Client 를 사용하고, 기본 구현체는 HttpUrlConnection (JDK 제공) 을 사용한다.
- HttpUrlConnection 이 기본 구현체인 이유 ?
    - JDK 에서 기본적으로 제공한다.
    - 단일 성능만 봤을때 가장 우수하다.
    - 대용량 트래픽을 다루는 서비스가 아니라면 문제 없다.
- 4.3 이후 부터 OkHttp 를 지원한다.
- MessageConverter 를 이용해 메세지 변환을 처리한다.

`주요 메소드`

| 메소드 | 설명 |
| --- | --- |
| delete() | 지정한 URL 리소스에 HTTP DELETE 요청을 수행한다. |
| exchange() | 지정된 HTTP 메소드를 URL 에 대해 실행하며, 응답 몸체와 연결되는 객체를 포함하는 ResponseEntity 를 반환 |
| execute() | 지정된 HTTP 메소드를 URL 에 대해 실행하며, 응답 몸체와 연결되는 객체를 반환 |
| getForEntity() | HTTP GET 요청을 전송하며, 응답 몸체와 연결되는 객체를 포함하는 ResponseEntity 를 반환 |
| getForObject() | HTTP GET 요청을 전송하며, 응답 몸체와 연결되는 객체를 반환 |
| headForHeaders() | HTTP HEAD 요청을 전송하며, 지정된 리소스 URL 의 HTTP 헤더를 반환 |
| optionsForAllow() | HTTP OPTIONS 요청을 전송하며, 지정된 URL 의 Allow 헤더를 반환 |
| patchForObject() | HTTP PATCH 요청을 전송하며, 응답 몸체와 연결되는 결과 객체를 반환 |
| postForEntity() | URL 에 데이터를 POST 하며, 응답 몸체와 연결되는 객체를 포함하는 ResponseEntity 를 반환 |
| postForLocation() | URL 에 데이터를 POST 하며, 새로 생성된 리소스의 URL 을 반환 |
| postForObject() | URL 에 데이터를 POST 하며, 응답 몸체와 연결되는 객체를 반환 |
| put() | 리소스 데이터를 지정한 URL 에 PUT |

## RestTemplate 사용하기
- 일반적으로 RestTemplate 을 Spring Bean 으로 등록한뒤, 이를 주입받아 사용한다.

```java
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class UserService implements UserDetailsService {
	private final UserRepository repository;
	private final PasswordEncoder passwordEncoder;
	private final ModelMapper mapper;

	private final RestTemplate restTemplate;
	private final Environment env;

	public UserDto getUserById(String userId) {
		User user = repository.findByUserId(userId)
			.orElseThrow(RuntimeException::new);
		UserDto userDto = mapper.map(user, UserDto.class);

		String orderUrl = String.format(env.getProperty("order_service.url"), userId);
		ResponseEntity<List<ResponseOrder>> ordersResponse = restTemplate.exchange(
			orderUrl,
			HttpMethod.GET,
			null,
			new ParameterizedTypeReference<List<ResponseOrder>>() {
			} // Super Type Token
		);

		List<ResponseOrder> responseOrders = ordersResponse.getBody();
		userDto.setOrders(responseOrders);
		return userDto;
	}
}
```

## RestTemplate With LoadBalancer
- Spring Cloud LoadBalancer (SCL) 은, Spring Cloud Ribbon 과 동일한 Client Size LoadBalancer
- 2018년 12월 이후 Ribbon 은 EOS 되었기에 SCL 을 사용하는 것이 좋다.

`Netflix Ribbon vs Spring Cloud LoadBalancer`
- Ribbon 은 Blocking 방식인 RestTemplate 에만 지원한다.
- 반면에 SCL 은 Non-Blocking 방식인 WebClient 를 지원함
- Ribbon 의 경우 RoundRobin, AvailabilityFilteringRule, WeightedResponseTimeRule 3가지 방식의 정책을 지원하고
- SCL 은 RoundRobin 과 Random 만을 지웒

`user-service.yml`

```yaml
order_serivce:
  url: http://ORDER-SERVICE/order-service/%s/order # LoadBalancer 를 사용 할 경우
  url: http://localhost:8000/order-service/%s/order # LoadBalancer 를 사용하지 않을 경우
```
- SCL (Ribbon) 을 사용할 경우 eureka service 에 등록된 마이크로서비스의 name 으로 접근이 가능해진다.