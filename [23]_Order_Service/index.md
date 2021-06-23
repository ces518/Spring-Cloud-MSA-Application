# Order Service

## 구성
- Spring Boot / Spring Cloud
- JPA (ORM)
- REST API
- H2

## features
- 사용자 별 상품 주문
- 사용자 별 주문 내역 조회

## APIs

| 기능 | URI | HTTP METHOD |
| --- | --- | --- |
| 사용자 별 상품 주문 | /order-service/{userId}/orders | POST |
| 사용자 별 주문 내역 조회 | /order-service/{userId}/orders | GET |

## 프로젝트 세팅

```yaml
server:
  port: 0 # 포트를 0번으로 지정하면 가용가능한 범위 내에서 랜덤한 포트로 실행됨

spring:
  application:
    name: order-service
  h2:
    console:
      enabled: true
      settings:
        web-allow-others: true
      path: /h2-console
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8761/eureka # Eureka 서버 정보
  instance:
    instance-id: ${spring.cloud.client.hostname}:${spring.application.instance_id:${random.value}}

logging:
  level:
    me.june.orderservice: DEBUG
```

## 기능 구현

```java
@Getter
@Setter
@Entity
@Table(name = "orders")
public class Order implements Serializable {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@Column(nullable = false)
	private String productId;

	@Column(nullable = false)
	private int qty;

	@Column(nullable = false)
	private int unitPrice;

	@Column(nullable = false)
	private int totalPrice;

	@Column(nullable = false)
	private String userId;

	@Column(nullable = false)
	private String orderId;

	@Column(nullable = false, updatable = false, insertable = false)
	@ColumnDefault("CURRENT_TIMESTAMP")
	private LocalDateTime createdAt;
}

@Data
public class OrderDto implements Serializable {
	private String productId;
	private int qty;
	private int unitPrice;
	private int totalPrice;

	private String orderId;
	private String userId;
}

@Data
public class RequestOrder {
	private String productId;
	private Integer qty;
	private Integer unitPrice;
}

@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ResponseOrder {
	private String productId;
	private int qty;
	private int unitPrice;
	private int totalPrice;
	private LocalDateTime createdAt;

	private String orderId;
}

public interface OrderRepository extends JpaRepository<Order, Long> {
	Optional<Order> findByOrderId(String orderId);
	Iterable<Order> findByUserId(String userId);
}

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class OrderService {
	private final OrderRepository repository;

	@Transactional
	public OrderDto createOrder(OrderDto orderDetail) {
		orderDetail.setOrderId(UUID.randomUUID().toString());
		orderDetail.setTotalPrice(orderDetail.getQty() * orderDetail.getUnitPrice());

		ModelMapper modelMapper = new ModelMapper();
		modelMapper.getConfiguration().setMatchingStrategy(MatchingStrategies.STRICT);

		Order order = modelMapper.map(orderDetail, Order.class);
		repository.save(order);

		OrderDto returnValue = modelMapper.map(order, OrderDto.class);
		return returnValue;
	}

	public OrderDto getOrderByOrderId(String orderId) {
		Order order = repository.findByOrderId(orderId)
				.orElseThrow(RuntimeException::new);
		OrderDto orderDto = new ModelMapper().map(order, OrderDto.class);
		return orderDto;
	}

	public Iterable<Order> getOrdersByUserId(String userId) {
		return repository.findByUserId(userId);
	}
}
```