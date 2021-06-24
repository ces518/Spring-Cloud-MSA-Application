# User Service Login

## APIs

| 기능 | URI | HTTP METHOD |
| --- | --- | --- |
| 로그인 | /user-service/login | POST |

## 기능 구현

```java
public class AuthenticationFilter extends UsernamePasswordAuthenticationFilter {

	@Override
	public Authentication attemptAuthentication(
		HttpServletRequest request,
		HttpServletResponse response
	) throws AuthenticationException {
		try {
			RequestLogin requestLogin = new ObjectMapper().readValue(request.getInputStream(), RequestLogin.class);
			return getAuthenticationManager().authenticate(
				new UsernamePasswordAuthenticationToken(
					requestLogin.getEmail(),
					requestLogin.getPassword(),
					new ArrayList<>()
				)
			);
		} catch (IOException e) {
			throw new RuntimeException(e);
		}
	}

	@Override
	protected void successfulAuthentication(
		HttpServletRequest request,
		HttpServletResponse response,
		FilterChain chain,
		Authentication authResult
	) throws IOException, ServletException {
		User user = (User)authResult.getPrincipal();
		logger.debug(String.format("{}, {}", user.getUsername(), user.getPassword()));
		super.successfulAuthentication(request, response, chain, authResult);
	}
}
```

> UsernamePasswordAuthenticationFilter 를 구현해서, username, password 에 대한 처리를 수행한다.

```java

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class UserService implements UserDetailsService {
	private final UserRepository repository;
	private final PasswordEncoder passwordEncoder;
	private final ModelMapper mapper;

	@Override
	public UserDetails loadUserByUsername(final String username) throws UsernameNotFoundException {
		User user = repository.findByEmail(username)
				.orElseThrow(() -> new UsernameNotFoundException(username));
		return new org.springframework.security.core.userdetails.User(
				user.getEmail(),
				user.getEncryptedPwd(),
				new ArrayList<>()
		);
	}
}
```

> UserDetailsService 를 구현

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	private UserService userService;

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.csrf().disable()
//				.authorizeRequests().antMatchers("/users/**").permitAll()
				.authorizeRequests().antMatchers("/**")
					.hasIpAddress("192.168.0.15")
				.and()
					.addFilter(getAuthenticationFilter())
				.headers().frameOptions().disable(); // 추가하지 않을경우 h2-console 이 접근이 되지 않는다.
	}

	@Bean
	public PasswordEncoder passwordEncoder() {
		return PasswordEncoderFactories.createDelegatingPasswordEncoder();
	}

	@Bean
	public AuthenticationFilter getAuthenticationFilter() throws Exception {
		AuthenticationFilter authenticationFilter = new AuthenticationFilter();
		authenticationFilter.setAuthenticationManager(authenticationManager());
		return authenticationFilter;
	}

	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth.userDetailsService(userService).passwordEncoder(passwordEncoder());
		super.configure(auth);
	}
}
```

> 구현한 filter 와, userDetailsService 를 Security 설정에 추가

## APIGateway 설정 추가

```yaml
spring:
  application:
    name: apigateway-service
  cloud:
    gateway:
      default-filters:
        - name: GlobalFilter
          args:
            baseMessage: Spring Cloud Gateway Global Filter
            preLogger: true
            postLogger: true
      routes:
        - id: first-service
          #          uri: http://localhost:8081/
          uri: lb://MY-FIRST-SERVICE # Discovery-Service 에 등록된 서비스 네임으로 접근
          predicates:
            - Path=/first-service/** # http://localhost:8081/first-serivce/** 형태로 그대로 전달됨을 주의..
          filters:
            #            - AddRequestHeader=first-request, first-request-header
            #            - AddResponseHeader=first-response, first-response-reader
            - CustomFilter
        - id: second-service
          #          uri: http://localhost:8082/
          uri: lb://MY-SECOND-SERVICE
          predicates:
            - Path=/second-service/**
          filters:
            #            - AddRequestHeader=second-request, second-request-header
            #            - AddResponseHeader=second-response, second-response-reader
            - CustomFilter
        #        - id: user-service
        #          uri: lb://USER-SERVICE
        #          predicates:
        #            - Path=/user-service/**
        - id: user-service # 로그인 요청
          uri: lb://USER-SERVICE
          predicates:
            - Path=/user-service/login
            - Method=POST
          filters:
            - RemoveRequestHeader=Cookie # 매번 새로운요청으로 인식하기 위해 Cookie 제거
            - RewritePath=/user-service/(?<segment>.*), /$\{segment}
        - id: user-service # 회원가입
          uri: lb://USER-SERVICE
          predicates:
            - Path=/user-service/users
            - Method=POST
          filters:
            - RemoveRequestHeader=Cookie # 매번 새로운요청으로 인식하기 위해 Cookie 제거
            - RewritePath=/user-service/(?<segment>.*), /$\{segment}
        - id: user-service # 로그인/회원가입 이후
          uri: lb://USER-SERVICE
          predicates:
            - Path=/user-service/**
            - Method=GET
          filters:
            - RemoveRequestHeader=Cookie # 매번 새로운요청으로 인식하기 위해 Cookie 제거
            - RewritePath=/user-service/(?<segment>.*), /$\{segment}
        - id: catalog-service
          uri: lb://CATALOG-SERVICE
          predicates:
            - Path=/catalog-service/**
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/order-service/**
```
- spring security 에서 제공하는 end-point 를 사용하기 위해 APIGateway 에서 RewritePath 처리를 해주어야한다.
