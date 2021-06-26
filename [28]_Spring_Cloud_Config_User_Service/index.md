# Spring Cloud Config - UserService

## User Service

`의존성 추가`

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

- spring cloud config, spring cloud bootstrap 의존성을 추가한다.
    - spring boot 2.4 이전 버전까지는 bootstrap.yml 을 기본으로 지원했지만 이후 부터는 application.yaml 파일을 이용해야한다.
    - bootstrap.yml 파일을 이용하는 방식 (legacy) 를 사용하려면 spring boot 애플리케이션 실행시 config-client.jar 를 넣어주거나 spring-cloud-starter-bootstrap 의존성을 추가해주는 방식 두가지중 하나를 선택해야한다.
    - 1. config-client.jar 를 넣어주는 방식을 선택한다면, spring.cloud.bootstrap.enable=true property 와 함께 넣어주어야 함
    - 2. spring-cloud-starter-bootstrap 의존성을 넣어주는 방식을 택한다면 별다른 설정이 필요없다.
    
- bootstrap 은 application.yaml 보다 높은 우선순위를 갖게 된다.


`bootstrap.yml`

```yaml
spring:
  cloud:
    config:
      uri: http://127.0.0.1:8888
      name: ecommerce
```

> profile 을 지정하지 않을 경우 기본적으로 default 로 요청하게 된다.

```yaml
server:
  port: 0 # 포트를 0번으로 지정하면 가용가능한 범위 내에서 랜덤한 포트로 실행됨

spring:
#  cloud:
#    bootstrap:
#      enabled: true
  application:
    name: user-service
  h2:
    console:
      enabled: true
      settings:
        web-allow-others: true
      path: /h2-console
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb
# h2 1.4.198 이후 버전으로는 봉나 문제로 자동으로 DB 를 생성하지 않는다.

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

greeting:
  message: Welcome to the Simple E-Commerce.


## Config Server 를 통해 Fetch 하도록 변경
#token:
#  expiration_time: 86400000
#  secret: user_token
```

`health_check API 변경`

```java
@RestController
@RequiredArgsConstructor
public class PingController {
	private final Environment env;

	@GetMapping("/health_check")
	public String healthCheck() {
		return String.format("It's Working in User Service %s" +
						"token.secret = %s, token.expire = $s",
				env.getProperty("local.server.port"),
				env.getProperty("token.secret"),
				env.getProperty("token.expiration_time")
		);
	}
}
```

## Config 정보 변경 반영
- 서버 재기동 (좋지 않은 방법)
- Actuator Refresh
    - Actuator : Application 상태 모니터링, Metric 수집을 위한 EndPoint 제공
- Spring Cloud Bus
    - Actuator 보다 효율적으로 정보를 사용할 수 있다.