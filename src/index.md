# Zuul Filter

```java
@Slf4j
@Component
public class ZuulLoggingFilter extends ZuulFilter {

	// 실제 필터의 동작 구현
	@Override
	public Object run() throws ZuulException {
		log.info("************* Printing logs :   ");
		RequestContext ctx = RequestContext.getCurrentContext();
		HttpServletRequest request = ctx.getRequest();
		log.info("************* {}", request.getRequestURI());
		return null;
	}

	@Override
	public String filterType() {
		return "pre";
	}

	@Override
	public int filterOrder() {
		return 1;
	}

	@Override
	public boolean shouldFilter() {
		return true;
	}
}
```
- ZuulFilter 추상 클래스를 구현하는 Bean 으로 등록되어 있어야한다.
    - run()
        - 실제 필터의 동작
    - filterType()
        - 필터의 타입
    - filterOrder()
        - 필터 우선 순위
    - shouldFilter()
        - 필터 사용 여부