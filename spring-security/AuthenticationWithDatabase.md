# 데이터베이스 기반 인증처리
---

## 인증처리 구조 다시보기

<p align=middle>
    <img src="./images/securityfilterchain.png" width=400>
</p>

1. DelegatingFilterProxy가 `FilterChainProxy`에 처리를 위임한다
2. FilterChainProxy는 가지고있던 SecurityFilterChain 중 클라이언트의 요청을 처리할 수 있는 SecurityFilterChain을 가져온다
3. 해당 SecurityFilterChain에서 인증 처리 필터를 호출해 인증처리를 수행한다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/142540771-b3d28271-1671-4f87-8d55-386abcf4a398.png width=600>
</p>

1. `UsernamePasswordAuthenticationFilter`에서 클라이언트가 전달한 username, password 정보를 기반으로 `UsernamePasswordAuthenticationToken`을 생성하여 `AuthenticationManager`에게 인증 처리를 요청한다.
2. `AuthenticationManager`(정확하게는 구현체인 `ProviderManager`)가 인증 처리를 수행한다.
3. 실패한 경우 후속 처리
4. 성공한 경우 후속 처리

### ProviderManager의 계층 구조
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/142551086-453cf873-9bf4-4dc9-934f-40057d0beb93.png width=600>
</p>

## DaoAuthenticationProvider

DaoAuthenticationProvider는 데이터베이스에서 사용자 정보를 조회하는 작업을 수행한다.

`authenticate` 메소드는 `AbstractUserDetailsAuthenticationProvider`에 구현되어있다.

```java
public abstract class AbstractUserDetailsAuthenticationProvider
		implements AuthenticationProvider, InitializingBean, MessageSourceAware {
    
    ...
	
    @Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		Assert.isInstanceOf(UsernamePasswordAuthenticationToken.class, authentication,
				() -> this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.onlySupports",
						"Only UsernamePasswordAuthenticationToken is supported"));
		String username = determineUsername(authentication);
		boolean cacheWasUsed = true;
		UserDetails user = this.userCache.getUserFromCache(username);
		if (user == null) {
			cacheWasUsed = false;
			try {
				user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
			}
			catch (UsernameNotFoundException ex) {
                ...
			}
			Assert.notNull(user, "retrieveUser returned null - a violation of the interface contract");
		}
		try {
			this.preAuthenticationChecks.check(user);
			additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
		}
		catch (AuthenticationException ex) {
			...
		}
		    ...
		return createSuccessAuthentication(principalToReturn, authentication, user);
	}
}
```

retrieveUser, additionalAuthenticationChecks, createSuccessAuthentication 세 가지 메서드가 `DaoAuthenticationProvider`에 구현되어있다. retrieveUser의 코드의 구현부만 살펴본다.

```java
public class DaoAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {
    ...
	@Override
	protected final UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication)
			throws AuthenticationException {
		prepareTimingAttackProtection();
		try {
			UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
			...
			return loadedUser;
		}
		...
	}
}
```

DaoAuthenticationProvider가 사용자 정보 조회를 UserDetailsService에 위임하는 것을 알 수 있다. 지금까지 메모리 기반 인증 처리를 수행할 때는 `InMemoryUserDetailsManager` 구현체를 사용했지만, 데이터베이스 기반 인증처리는 `JdbcDaoImpl`을 사용한다.

## JdbcDaoImpl
```java
public class JdbcDaoImpl extends JdbcDaoSupport implements UserDetailsService, MessageSourceAware {

    public static final String DEF_USERS_BY_USERNAME_QUERY = "select username,password,enabled "
        + "from users "
        + "where username = ?";

	public static final String DEF_AUTHORITIES_BY_USERNAME_QUERY = "select username,authority "
			+ "from authorities "
			+ "where username = ?";

	public static final String DEF_GROUP_AUTHORITIES_BY_USERNAME_QUERY = "select g.id, g.group_name, ga.authority "
			+ "from groups g, group_members gm, group_authorities ga "
			+ "where gm.username = ? " + "and g.id = ga.group_id " + "and g.id = gm.group_id";

	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		List<UserDetails> users = loadUsersByUsername(username);
		if (users.size() == 0) {
            ...
		}
		UserDetails user = users.get(0); // contains no GrantedAuthority[]
		Set<GrantedAuthority> dbAuthsSet = new HashSet<>();
		if (this.enableAuthorities) {
			dbAuthsSet.addAll(loadUserAuthorities(user.getUsername()));
		}
		if (this.enableGroups) {
			dbAuthsSet.addAll(loadGroupAuthorities(user.getUsername()));
		}
		List<GrantedAuthority> dbAuths = new ArrayList<>(dbAuthsSet);
		addCustomAuthorities(user.getUsername(), dbAuths);
		if (dbAuths.size() == 0) {
			...
		}
		return createUserDetails(username, user, dbAuths);
	}
    
    protected List<UserDetails> loadUsersByUsername(String username) {
		// @formatter:off
		RowMapper<UserDetails> mapper = (rs, rowNum) -> {
			String username1 = rs.getString(1);
			String password = rs.getString(2);
			boolean enabled = rs.getBoolean(3);
			return new User(username1, password, enabled, true, true, true, AuthorityUtils.NO_AUTHORITIES);
		};
		// @formatter:on
		return getJdbcTemplate().query(this.usersByUsernameQuery, mapper, username);
	}

    protected List<GrantedAuthority> loadUserAuthorities(String username) {
		return getJdbcTemplate().query(this.authoritiesByUsernameQuery, new String[] { username }, (rs, rowNum) -> {
			String roleName = JdbcDaoImpl.this.rolePrefix + rs.getString(2);
			return new SimpleGrantedAuthority(roleName);
		});
	}

    protected List<GrantedAuthority> loadGroupAuthorities(String username) {
		return getJdbcTemplate().query(this.groupAuthoritiesByUsernameQuery, new String[] { username },
				(rs, rowNum) -> {
					String roleName = getRolePrefix() + rs.getString(3);
					return new SimpleGrantedAuthority(roleName);
				});
	}
}
```