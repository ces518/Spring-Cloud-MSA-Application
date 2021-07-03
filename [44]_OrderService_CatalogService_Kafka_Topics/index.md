# OrderService CatalogService Kafka Topics

## Orders -> Catalogs 데이터 동기화
- 각 마이크로 서비스들은 독립된 데이터베이스를 가지고 있음
- 주문이 들어오면 Catalog Service 의 재고에서 - 처리를 해주어야 한다.
- 둘 사이의 데이터 동기화를 위해 카프카를 사용한다.

## Catalog Service

`의존성 추가`

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

`카프카 설정`

```java
@Configuration
@EnableKafka
public class KafkaConsumerConfig {

	@Bean
	public ConsumerFactory<String, String> consumerFactory() {
		Map<String, Object> properties = new HashMap<>();
		properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:9092");
		properties.put(ConsumerConfig.GROUP_ID_CONFIG, "consumerGroupId");
		properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
		properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

		return new DefaultKafkaConsumerFactory<>(properties);
	}

	@Bean
	public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
		ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory = new ConcurrentKafkaListenerContainerFactory<>();
		kafkaListenerContainerFactory.setConsumerFactory(consumerFactory());

		return kafkaListenerContainerFactory;
	}
}
```
- 주의할점은 **kafkaListenerContainerFactory** 라는 이름의 빈으로 등록해야 한다.
- 만약 그렇지 않을 경우 자동 설정 빈이 등록된다.
    - ConsumerConfig.GROUP_ID_CONFIG 관련 설정이 등록되지 않아 예외가 발생한다.

`컨슈머 설정`

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class KafkaConsumer {
	private final CatalogRepository catalogRepository;

	@KafkaListener(topics = "example-catalog-topic")
	public void updateQty(String kafkaMessage) {
		log.info("Kafka Message : -> ", kafkaMessage);

		Map<Object, Object> map = new HashMap<>();
		ObjectMapper mapper = new ObjectMapper();
		try {
			map = mapper.readValue(kafkaMessage, new TypeReference<Map<Object, Object>>(){});
		} catch (JsonProcessingException e) {
			e.printStackTrace();
		}
		String productId = (String) map.get("productId");

		Catalog catalog = catalogRepository.findByProductId(productId).orElseThrow();
		int qty = (int) map.get("qty");
		catalog.setStock(catalog.getStock() - qty);

		catalogRepository.save(catalog);
	}
}
```

## OrderService

`의존성 추가`

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

`카프카 설정`

```xml
@Configuration
@EnableKafka
public class KafkaProducerConfig {

	@Bean
	public ProducerFactory<String, String> producerFactory() {
		Map<String, Object> properties = new HashMap<>();
		properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:9092");
		properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
		properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

		return new DefaultKafkaProducerFactory<>(properties);
	}

	@Bean
	public KafkaTemplate<String, String> kafkaTemplate() {
		return new KafkaTemplate<>(producerFactory());
	}
}
```

```java
@RestController
@RequestMapping("/order-service")
@RequiredArgsConstructor
public class OrderController {
	private final OrderService service;
	private final KafkaProducer kafkaProducer;

	@PostMapping("/{userId}/orders")
	public ResponseEntity<ResponseOrder> createOrder(
			@PathVariable String userId,
			@RequestBody RequestOrder orderDetail) {
		ModelMapper mapper = new ModelMapper();
		mapper.getConfiguration().setMatchingStrategy(MatchingStrategies.STRICT);

		OrderDto orderDto = mapper.map(orderDetail, OrderDto.class);
		orderDto.setUserId(userId);
		service.createOrder(orderDto);

		ResponseOrder responseOrder = mapper.map(orderDto, ResponseOrder.class);

		/* Send To Kafka */
		kafkaProducer.send("example-catalog-topic", orderDto);
		/**/

		return ResponseEntity.status(HttpStatus.CREATED).body(responseOrder);
	}

	@GetMapping("/{userId}/orders")
	public ResponseEntity<List<ResponseOrder>> getOrder(@PathVariable String userId) {
		Iterable<Order> orders = service.getOrdersByUserId(userId);

		List<ResponseOrder> result = new ArrayList<>();
		orders.forEach(v -> {
			result.add(new ModelMapper().map(v, ResponseOrder.class));
		});
		return ResponseEntity.ok(result);
	}
}
```
- 주문이 완료되면 **example-catalog-topic** 으로 메세지를 발행한다.

> 주문완료시, **example-catalog-topic** 에 발행된 메세지를 CatalogService 가 Consume 해서, 수량을 차감하는 구조 