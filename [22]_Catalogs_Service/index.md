# User Service

## 구성
- Spring Boot / Spring Cloud
- JPA (ORM)
- REST API
- H2

## features
- 상품 목록 조회

## APIs

| 기능 | URI | HTTP METHOD |
| --- | --- | --- |
| 상품 목록 조회 | /catalog-service/catalogs | GET |

## 프로젝트 세팅

```yaml
server:
  port: 0 # 포트를 0번으로 지정하면 가용가능한 범위 내에서 랜덤한 포트로 실행됨

spring:
  application:
    name: catalog-service
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
      ddl-auto: create-drop
    show-sql: true
    generate-ddl: true
    defer-datasource-initialization: true # 기본적으로 hibernate schema 생성 이전에 data.sql 을 읽어들인다.
    # hibernate ddl 이후에 호출하려면 위 옵션을 추가해주어야한다.

# Eureka 등록 및 외부 검색 허용 설정
# Eureka Server 의 기본 표기법 = ${PORT}:${Spring.application.name}:${yml의 port} 로 인스턴스를 표기한다
# Random Port 로 기동할 경우 0 port 에 대해서만 표기가 된다.. => 이는 표기법에 대한 문제
# instance-id 에 대한 설정이 필요함
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
    me.june.catalogservice: DEBUG
```

## 기능 구현

```java
@Getter
@Setter
@Entity
@Table(name = "catalogs")
@EqualsAndHashCode(of = "id")
public class Catalog implements Serializable {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@Column(nullable = false)
	private String productId;

	@Column(nullable = false)
	private String productName;

	@Column(nullable = false)
	private Integer stock;

	@Column(nullable = false)
	private Integer unitPrice;

	@Column(nullable = false, updatable = false, insertable = false)
	@ColumnDefault("CURRENT_TIMESTAMP")
	private LocalDateTime createdAt;
}

@Data
public class CatalogDto implements Serializable {
	private String productId;
	private Integer qty;
	private Integer unitPrice;
	private Integer totalPrice;

	private String orderId;
	private String userId;
}

@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ResponseCatalog {
	private String productId;
	private String productName;
	private Integer stock;
	private Integer unitPrice;
	private LocalDateTime createdAt;
}

@RestController
@RequestMapping("/catalog-service")
@RequiredArgsConstructor
public class CatalogController {
	private final CatalogService service;

	@GetMapping("/catalogs")
	public ResponseEntity<List<ResponseCatalog>> getCatalogs() {
		Iterable<Catalog> catalogs = service.getCatalogs();

		List<ResponseCatalog> result = new ArrayList<>();
		catalogs.forEach(v -> {
			result.add(new ModelMapper().map(v, ResponseCatalog.class));
		});
		return ResponseEntity.ok(result);
	}
}

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class CatalogService {
	private final CatalogRepository repository;

	public Iterable<Catalog> getCatalogs() {
		return repository.findAll();
	}
}

public interface CatalogRepository extends JpaRepository<Catalog, Long> {
	Optional<Catalog> findByProductId(String productId);
}
```