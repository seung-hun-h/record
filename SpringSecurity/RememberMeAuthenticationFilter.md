# Remember-me
---
Remember-me 또는 Persistent-login 인증은 웹 사이트가 세션 사이에서 신원을 기억하고 있는 것을 말한다. 클라이언트에 Remember-me 쿠키를 보내고, 다음 세션에서 해당 쿠키가 감지될 경우 자동으로 로그인을 성공하게 한다.

스프링 시큐리티에서는 Remember-me를 구현하기 위해 두 가지 방법(Hook)을 사용한다. 하나는 쿠키 기반 토큰의 보안을 위해 **해싱(hashing)**을 사용하는 것이고, 다른 하나는 생성된 토큰을 저장하기 위해 **데이터베이스** 또는 **영구 저장 메커니즘**을 사용하는 것이다.

두 가지 구현 방법 모두 `UserDetailsService`를 사용한다.

## Simple Hash-Based Token Approach
인증이 성공하면 클라언트에 쿠키를 보낸다. 쿠키의 구성은 다음과 같다.

```txt
base64(username + ":" + expirationTime + ":" +
md5Hex(username + ":" + expirationTime + ":" password + ":" + key))

username:          As identifiable to the UserDetailsService
password:          That matches the one in the retrieved UserDetails
expirationTime:    The date and time when the remember-me token expires, expressed in milliseconds
key:               A private key to prevent modification of the remember-me token
```

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/141998152-a8fd6ca7-1f4f-41fe-b0b0-2076cabf74a4.png width=500>
</p>

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/141998277-3b15047c-d954-40b5-a79a-92cfafcc060f.png width=500>
</p>


## Persistent Token Approach
데이터베이스와 같은 영구 데이터 저장소를 사용하는 방법이다. 해당 방법을 사용하기 위해서는 `persistent_logins` 테이블을 생성해야한다.

```SQL
create table persistent_logins (username varchar(64) not null,
								series varchar(64) primary key,
								token varchar(64) not null,
								last_used timestamp not null)
```

# Remember-Me Interfaces and Implementations
---
Remember-Me가 사용되는 인터페이스와 그 구현체들은 아래와 같다.

- AbstractAuthenticationProcessingFilter
  - UsernamePasswordAuthenticationFilter
- RememberMeServices
  - AbstractRememberMeServices
    - TokenBasedRememberMeServices
    - PersistentTokenBasedRememberMeServices


<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/141999452-3cc676df-95b2-43c2-aa9a-630f484a2a84.png width=500>
</p>

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/141999536-85bd50ea-b476-4be8-a9d3-999171164168.png width=500>
</p>


## Set Cookie - 최초 로그인

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/142002335-f942af77-07f0-4653-aaa7-91d495fd9c62.png width=500>
</p>

- username과 password가 일치하면 `AbstractAuthenticationProcessingFilter`의 `successfulAuthentication()`가 호출된다
- `successfulAuthentication()`에서 다시 `AbstractRememberMeServices`의 `loginSuccess()`가 호출된다.
- 그리고 `TokenBasedRememberMeServices`에 재정의된 `onLoginSuccess()`가 호출되어 쿠키에 Remember-me 관련 데이터가 저장된다.


## Auto Login

RememberMeAuthenticationFilter에서 로직이 동작하고 RememberMeService와 RememberMeAuthenticationProvider에서 세부 로직이 동작한다. RememberMeService에서 자동 로그인을 시도하고 Authentication 인스턴스를 받아오면 RememberMeAuthenticationProvider에서 인증 절차를 수행한다.


### RememberMeAuthenticationFilter

```java
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
        throws IOException, ServletException {
    if (SecurityContextHolder.getContext().getAuthentication() != null) {
        ...
        chain.doFilter(request, response);
        return;
    }
    Authentication rememberMeAuth = this.rememberMeServices.autoLogin(request, response); // 1
    if (rememberMeAuth != null) {
        // Attempt authenticaton via AuthenticationManager
        try {
            rememberMeAuth = this.authenticationManager.authenticate(rememberMeAuth); // 2
            // Store to SecurityContextHolder
            SecurityContextHolder.getContext().setAuthentication(rememberMeAuth);
            onSuccessfulAuthentication(request, response, rememberMeAuth); 
            ...
            if (this.eventPublisher != null) {
                this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(
                        SecurityContextHolder.getContext().getAuthentication(), this.getClass()));
            }
            if (this.successHandler != null) {
                this.successHandler.onAuthenticationSuccess(request, response, rememberMeAuth); 
                return;
            }
        }
        catch (AuthenticationException ex) { // 3
            ...
            this.rememberMeServices.loginFail(request, response);
            onUnsuccessfulAuthentication(request, response, ex);
        }
    }
    chain.doFilter(request, response);
}
```
1. RememberMeService에서 `autoLogin()`을 호출한다.
   - `autoLogin()`가 성공하면 RememberMeAuthenticationToken을 반환하고 그렇지 않으면 null을 반환한다
2. RememberMeAuthenticationProvider에서 인증을 수행한다
   - remember-me 설정 시 입력했던 키(안했으면 랜덤)와 Authentication 인스턴스의 키를 비교하여 같으면 성공한다
3.  `AuthenticationException`이 발생한 경우 RememberMeService에서 `loginFail()`을 호출하여, 인증 실패 처리를 수행한다.

### RememberMeService
`AbstractRememberMeService`에서 로그인과 관련된 공통된 로직인 `autoLogin()`이 동작하고, 구현체들에게서 재정의된 `processAutoLoginCookie()`이 동작한다.

```java
public final Authentication autoLogin(HttpServletRequest request, HttpServletResponse response) {
    String rememberMeCookie = extractRememberMeCookie(request); // 1
    ...
    try {
        String[] cookieTokens = decodeCookie(rememberMeCookie);
        UserDetails user = processAutoLoginCookie(cookieTokens, request, response); // 2
        this.userDetailsChecker.check(user);
        this.logger.debug("Remember-me cookie accepted");
        return createSuccessfulAuthentication(request, user); // 3
    }
    catch (Exception ex) {
        ...
    }
    cancelCookie(request, response);
    return null; // 4
}
```

1. 쿠키에서 Remember-me와 관련한 정보를 추출한다
2. 쿠키 정보를 가지고 자동 로그인을 시도한다
3. 성공하면 RememberMeAuthenticationToken을 반환한다
4. 실패하면 null을 반환한다

```java
// TokenBasedRememberMeServices
@Override
protected UserDetails processAutoLoginCookie(String[] cookieTokens, HttpServletRequest request,
        HttpServletResponse response) {
    ...
    String expectedTokenSignature = makeTokenSignature(tokenExpiryTime, userDetails.getUsername(),
            userDetails.getPassword());
    if (!equals(expectedTokenSignature, cookieTokens[2])) {
        throw new InvalidCookieException("Cookie token[2] contained signature '" + cookieTokens[2]
                + "' but expected '" + expectedTokenSignature + "'");
    }
    return userDetails;
}

//PersistentTokenBasedRememberMeServices
@Override
protected UserDetails processAutoLoginCookie(String[] cookieTokens, HttpServletRequest request,
        HttpServletResponse response) {
        ...
    PersistentRememberMeToken token = this.tokenRepository.getTokenForSeries(presentedSeries);
        ...
    // We have a match for this user/series combination
    if (!presentedToken.equals(token.getTokenValue())) {
        // Token doesn't match series value. Delete all logins for this user and throw
        // an exception to warn them.
        this.tokenRepository.removeUserTokens(token.getUsername());
        ...
    }
        ...
    PersistentRememberMeToken newToken = new PersistentRememberMeToken(token.getUsername(), token.getSeries(),
            generateTokenData(), new Date());
    try {
        this.tokenRepository.updateToken(newToken.getSeries(), newToken.getTokenValue(), newToken.getDate());
        addCookie(newToken, request, response);
    }
    catch (Exception ex) {
        ...
    }
    return getUserDetailsService().loadUserByUsername(token.getUsername());
}
```
**TokenBased**
- username + password + expiry time + key를 바탕으로 Expected Token Signature를 만들어 실제와 비교한다

**PersistentTokenBased**
- 쿠키에 저장된 series 값을 가지고 토큰을 데이터베이스에서 가져오고, 실제 토큰과 비교한다
- 로그인이 성공할 때 마다 series 값을 갱신하고 데이터베이스에 반영한다.

### RememberMeAuthenticationProvider

```java
@Override
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    if (!supports(authentication.getClass())) {
        return null;
    }
    if (this.key.hashCode() != ((RememberMeAuthenticationToken) authentication).getKeyHash()) {
        throw new BadCredentialsException(this.messages.getMessage("RememberMeAuthenticationProvider.incorrectKey",
                "The presented RememberMeAuthenticationToken does not contain the expected key"));
    }
    return authentication;
}
```
RememberMeAuthenticationProvider에서는 인자로 전달받은 RememberMeAuthenticationToken의 키 해시 값과 자신의 키 해시 값이 같은지만 비교한다.

### 자동 로그인을 원하지 않을 때
어떠한 자원에 접근할 때는 Remember-me 기능을 활용한 자동 로그인이 아니라 아이디/비밀번호 기반으로 명시적인 로그인이 필요하기도 하다.

Remember-Me 기반 인증 결과는 `RememberMeAuthenticationToken`을 반환하고 로그인 아이디/비밀번호 기반 인증 결과는 `UsernamePasswordAuthenticationToken`을 반환한다.

따라서 동일하게 인증된 사용자라도 권한을 분리할 수 있다. 이를 위해서 `isFullyAuthenticated()`를 사용할 수 있다.

```java
@Override
public final boolean isFullyAuthenticated() {
	return !this.trustResolver.isAnonymous(this.authentication)
			&& !this.trustResolver.isRememberMe(this.authentication);
}
```

다음과 같이 사용할 수 있다.

```java
 protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/me").hasAnyRole("USER", "ADMIN")
                .antMatchers("/admin").access("isFullyAuthenticated() and hasRole('ADMIN')")

        ...
```

---
참고
- https://docs.spring.io/spring-security/reference/servlet/authentication/rememberme.html#page-title
- https://www.baeldung.com/spring-security-remember-me