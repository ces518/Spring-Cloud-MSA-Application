# User Service Jwts

## Jwt 인증 

`전통적인 인증 방식`
- form 인증 방식을 많이 사용
- session 이용해 사용자 정보를 유지한다.

> 세션과 쿠키는 모바일 앱 에서 유효하게 사용하기 힘들다. (FE/BE 간의 분리가 힘듦)
> 대부분 서버단에서 구현된 기술 (JSP 등..) 을 이용할때 사용한 전통적인 방식

`Token 기반 인증`
- Jwt 도 포함됨
- 서버에서 인증 절차를 거친 뒤, Token 을 발급한다.
- 인증된 후 이 토큰을 이용해 인증을 한다.
- 일반적으로 Bearer Token 방식을 이용함
    - Authorization: Bearer ...

`Jwt`
- 토큰 포맷
- https://jwt.io/
- stateless
- CDN 과 인증 처리가 가능함
- No Cookie-Session (No CSRF)
- 지속적인 토큰 저장

## Jwt 를 이용한 토큰 발행

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

```java

public class AuthenticationFilter extends UsernamePasswordAuthenticationFilter {

	@Autowired
	private UserService userService;

	@Autowired
	private Environment env;

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
		UserDto userDetails = userService.getUserDetailsByEmail(user.getUsername());

		String token = Jwts.builder()
				.setSubject(userDetails.getUserId())
				.setExpiration(new Date(
						System.currentTimeMillis() + Long.parseLong(env.getProperty("token.expiration_time"))
					)
				)
				.signWith(SignatureAlgorithm.HS512, env.getProperty("token.secret"))
				.compact();
		response.addHeader("token", token);
		response.addHeader("userId", userDetails.getUserId() );
		logger.debug(String.format("{}, {}", user.getUsername(), user.getPassword()));
	}
}
```
- 인증 성공시 header 에 인증 토큰 정보를 함께 내려준다.