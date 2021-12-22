# RequestCacheAwareFilter
---

현재 요청을 처리할 때 캐시되어 있는 요청이 있는 경우 요청을 처리해주는 필터이다. 예를 들어, 접근 권한이 있는 URL(www.example.com/users/detail)을 요청한 경우 로그인 페이지로 리다이렉트 된 후 로그인이 성공하면 캐시된 요청 URL(www.example.com/users/detail)이 다시 요청된다.

`RequestCacheAwareFilter`의 소스 코드는 아래와 같은데 별건 없고 저장된 요청(wrappedSavedRequest)이 있는 경우에는 해당 요청을 가지고 필터링 작업을 수행한다.

```java
public class RequestCacheAwareFilter extends GenericFilterBean {

	private RequestCache requestCache;

    ...

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest wrappedSavedRequest = this.requestCache.getMatchingRequest((HttpServletRequest) request,
				(HttpServletResponse) response);
		chain.doFilter((wrappedSavedRequest != null) ? wrappedSavedRequest : request, response);
	}

}
```

## 실행 흐름

접근 권한이 필요한 자원에 요청을 보낼 경우 인증 후 인가처리를 위해 요청이 `FilterSecurityInterceptor`까지 도달하고 `AccessDicisionManager`에 의해서 `AccessDeniedException`이 발생한다.

`AccessDeniedException`는 다시 `ExceptionTranslationFilter`까지 타고 올라가서 `sendStartAuthentication()`가 호출되어 요청이 저장되고 `LoginUrlAuthenticationEntryPoint`의 `commence()`가 호출되어 로그인 페이지로 리다이렉팅 된다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/141984976-cf8f169c-f4af-4fbd-955b-9cd362667055.png
    width=500>
</p>

### ExceptionTranslationFilter

**handleAccessDeniedException()**

```java
private void handleAccessDeniedException(HttpServletRequest request, HttpServletResponse response,
        FilterChain chain, AccessDeniedException exception) throws ServletException, IOException {
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
    boolean isAnonymous = this.authenticationTrustResolver.isAnonymous(authentication);
    if (isAnonymous || this.authenticationTrustResolver.isRememberMe(authentication)) {
        ...
        // 여기
        sendStartAuthentication(request, response, chain,
                new InsufficientAuthenticationException(
                        this.messages.getMessage("ExceptionTranslationFilter.insufficientAuthentication",
                                "Full authentication is required to access this resource")));
    }
    else {
        ...
    }
}
```

**sendStartAuthentication()**

```java
    	protected void sendStartAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,
			AuthenticationException reason) throws ServletException, IOException {
		SecurityContextHolder.getContext().setAuthentication(null);
		this.requestCache.saveRequest(request, response); // 요청 저장
		this.authenticationEntryPoint.commence(request, response, reason); // 로그인 폼으로 리다이렉션
	}
```

---
참고
- https://kangwoojin.github.io/programing/spring-security-basic-request-cache-aware-filter/