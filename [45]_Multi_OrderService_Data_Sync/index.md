# Multi OrderService Data Sync

## Multi Instance 환경에서 데이터 싱크 문제
- Order Service Instance 가 여러개 일 경우, 각 인스턴스가 개별 데이터베이스를 가지고 있기 때문에 데이터 불일치 발생
- Kafka Connector + 단일 DB 조합으로 사용

## OrderProducer

```java
@Data
@AllArgsConstructor
public class KafkaOrderDto implements Serializable {
	private Schema schema;
	private Payload payload;
}

@Data
@Builder
public class Schema {
	private String type;
	private List<Field> fields;
	private boolean optional;
	private String name;
}

@Data
@AllArgsConstructor
public class Field {
	private String type;
	private boolean optional;
	private String field;
}

@Data
@Builder
public class Payload {
	@JsonProperty("order_id")
	private String orderId;

	@JsonProperty("user_id")
	private String userId;

	@JsonProperty("product_id")
	private String productId;

	private int qty;

	@JsonProperty("unit_price")
	private int unitPrice;

	@JsonProperty("total_price")
	private int totalPrice;
}

@Slf4j
@Component
@RequiredArgsConstructor
public class OrderProducer {
	private final KafkaTemplate<String, String> kafkaTemplate;

	List<Field> fields = List.of(
		new Field("string", true, "order_id"),
		new Field("string", true, "user_id"),
		new Field("string", true, "product_id"),
		new Field("int32", true, "qty"),
		new Field("int32", true, "unit_price"),
		new Field("int32", true, "total_price")
	);

	Schema schema = Schema.builder()
		.type("struct")
		.fields(fields)
		.optional(false)
		.name("orders")
		.build();

	public OrderDto send(String topic, OrderDto orderDto) {
		Payload payload = Payload.builder()
			.orderId(orderDto.getOrderId())
			.productId(orderDto.getProductId())
			.userId(orderDto.getUserId())
			.qty(orderDto.getQty())
			.unitPrice(orderDto.getUnitPrice())
			.totalPrice(orderDto.getTotalPrice())
			.build();

		KafkaOrderDto kafkaOrderDto = new KafkaOrderDto(schema, payload);

		ObjectMapper mapper = new ObjectMapper();
		try {
			String body = mapper.writeValueAsString(kafkaOrderDto);
			kafkaTemplate.send(topic, body);
			log.info("Order Producer send data from the Order Service : ", orderDto);
		} catch (JsonProcessingException e) {
			e.printStackTrace();
		}
		return orderDto;
	}
}
```
- Kafka Connect 의 Topic 으로 보낼 Producer 를 정의하고, Kafka JDBC Connect 의 format DTO 정의

## Order Sink Connector 등록

```shell
curl -X "POST" "http://localhost:8083/connectors" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "name": "my-order-sink-connect",
  "config": {
    "topics": "orders",
    "connection.password": "root!",
    "table.name.format": "orders",
    "delete.enabled": "false",
    "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "auto.create": "true",
    "tasks.max": "1",
    "auto.evolve": "true",
    "connection.user": "root",
    "connection.url": "jdbc:mysql://localhost:3306/mydb"
  }
}'
```

> OrderService 를 통해 주문이 발생하면, **orders** Topic 을 통해 주문 정보 메세지가 발행되고, Order-Sink-Connector 에 의해 MariaDB 에 저장되는 구조