# User Service

## 구성
- Spring Boot / Spring Cloud
- JPA (ORM)
- REST API
- H2

## features
- 회원 등록
- 로그인
- 상세
- 수정/삭제
- 상품 주문
- 주문 내역 확인

## APIs

| 기능 | URI | HTTP METHOD |
| --- | --- | --- |
| 등록 | /user-service/users | POST |
| 조회 | /user-service/users | GET |
| 정보 조회 | /user-service/users/{id} | GET |
| 동작 확인 | /user-service/users/health_check | GET |
| 환영 메세지 | /user-service/welcome | GET |

## 의존성 추가 및 설정

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.196</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>2.3.8</version>
</dependency>
```

`ModelMapperConfig`
```java
@Configuration
public class ModelMapperConfig {

	@Bean
	public ModelMapper modelMapper() {
		ModelMapper modelMapper = new ModelMapper();
		modelMapper.getConfiguration().setMatchingStrategy(MatchingStrategies.STRICT);
		return modelMapper;
	}
}
```

## 사용자 등록 구현

```java
@Getter
@Setter
@Entity
@Table(name = "users")
@EqualsAndHashCode(of = "id")
public class User implements Serializable {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@Column(nullable = false)
	private String email;

	@Column(nullable = false)
	private String name;

	@Column(nullable = false)
	private String userId;

	@Column(nullable = false)
	private String encryptedPwd;
}

@Getter
public class UserRequest {
	@NotNull(message = "Email cannot be null")
	@Size(min = 2, message = "Email not be less than two characters")
	@Email
	private String email;

	@NotNull(message = "Password cannot be null")
	@Size(min = 8, message = "Password must be equal or greater than 8 characters and less then 16 characters")
	private String pwd;

	@NotNull(message = "Name cannot be null")
	@Size(min = 2, message = "Name not be less than two characters")
	private String name;
}


@Data
public class UserDto {
	private String email;
	private String pwd;
	private String name;
	private String userId;
	private LocalDateTime createdAt;

	private String encryptedPwd;
}

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class UserService {
	private final UserRepository repository;
	private final ModelMapper mapper;

	@Transactional
	public UserDto createUser(final UserRequest request) {
		User entity = mapper.map(request, User.class);
		entity.setUserId(UUID.randomUUID().toString());
		entity.setEncryptedPwd("encrypted password");
		User savedUser = repository.save(entity);
		return mapper.map(savedUser, UserDto.class);
	}
}

@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
public class UserController {
	private final Environment env;
	private final UserService userService;

	@GetMapping("/welcome")
	public String welcome() {
		return env.getProperty("greeting.message");
	}

	@PostMapping
	public ResponseEntity<UserDto> createUser(@RequestBody final UserRequest request) {
		return ResponseEntity.status(HttpStatus.CREATED).body(userService.createUser(request));
	}
}
```