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

