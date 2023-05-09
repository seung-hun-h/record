## 스프링 트랜잭션
- 다른 데이터 접근 기술을 사용하면 트랜잭션 처리 방식에 차이가 발생할 수 있다

```Java
// jdbc
public static void main(String[] args) {
	Connection conn = dataSource.getConnection();
	try {
		conn.setAutoCommit(false);
		...
		conn.commit();
	} catch (Exception e) {
		conn.rollback();
		throw new IllegalStateException(e);
	} finally {
		release(conn);
	}
}

// jpa
public static void main(String[] args) {
	EntityManagerFactory emf = Persistence.createEntityManagerFactory(...);
	EntityManager em = emf.createEntityManager();
	EntityTransaction tx = em.getTransaction();
	
	try {
		tx.begin();
		...
		tx.commit();
	} catch (Exception e) {
		tx.rollback();
		throw new IllegalStateException(e);
	} finally {
		em.close();
	}
	emf.close();
}
```

- 스프링은 이러한 문제를 해결하기 위해 `PlatformTransactionManager`라는 트랜잭션 추상화를 제공한다

#### PlatformTransactionManager
```Java

public interface PlatformTransactionManager extends TransactionManager {

  TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
		  throws TransactionException;

  void commit(TransactionStatus status) throws TransactionException;

  void rollback(TransactionStatus status) throws TransactionException;
}
```

<img width="650" alt="Screenshot 2023-05-10 at 6 27 52 AM" src="https://github.com/seung-hun-h/record/assets/60502370/a71b4701-9447-4541-a676-f31523f9a0e3">
- 스프링은 트랜잭션 추상화 뿐 아니라, 주로 사용하는 데이터 접근 기술에 대한 트랜잭션 매니저의 구현체도 제공한다
	- 스프링 5.3부터는 JDBC 트랜잭션을 관리할 때 `DataSourceTransactionManager`를 상속받아서 약간의 기능을 확장한 `JdbcTransactionManager`를 제공한다

### 스프링 트랜잭션 사용 방식
#### 선언적 트랜잭션 관리
- `@Transactional` 애노테이션 하나만 선언해서 편리하게 트랜잭션을 적용할 수 있다
- 스프링 AOP을 적용해 프록시 방식으로 동작한다
- 주로 이 방법을 사용한다

#### 프로그래밍 방식 트랜잭션 관리
- 트랜잭션 매니저 혹은 트랜잭션 템플릿 등을 사용해서 트랜잭션 관련 코드를 직접 작성하는 방식이다

#### 선언적 트랜잭션 관리 동작 방식
<img width="650" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/0b34dabe-4d6f-42f4-9184-98039b6282d3">
- 커넥션에 `conn.setAutoCommit(false)`를 지정하면서 시작한다
- 같은 트랜잭션을 유지하려면 같은 데이터베이스 커넥션을 사용해야 한다
	- 트랜잭션 동기화 매니저를 내부적으로 사용한다

#### @Transactional 사용
```Java
@Transactional(readOnly=true)
public class BizService {
	public Object getSomething() {
		...
	}

	@Transactional
	public void createSomething() {
		...
	}
}
```

- `getSomething()`은 데이터를 조회하는 로직만 가지고 있고, `createSomething()`은 데이터를 생성하는 로직을 가지고 있다
- `@Transactional`의 적용 우선순위는 더 구체적인 것이 우선한다
	- 클래스보다 메서드가 더 구체적이므로 `createSomething()`는 `@Transactional`이 적용된다

- 인터페이스에도 `@Transactional`을 적용할 수 있다
```Java
@Transactional(readOnly=true)
public interface SuperBizService {
	Object getSomething();
	
	@Transactional
	void createSomething();
}
```
- 이때도 더 구체적인 것이 우선한다
1. 클래스의 메서드
2. 클래스의 타입
3. 인터페이스의 메서드
4. 인터페이스의 타입

#### @Transactional 사용시 유의점
- 스프링 AOP를 적용하여  프록시 형태로 트랜잭션을 관리하기 때문에 **내부 호출**문제가 발생할 수 있다
```Java
public class BizService {
	public Object external() {
		...
		internal(); // 프록시가 아닌 실제 객체의 메서드를 호출해 트랜잭션이 적용되지 않음
		...
	}

	@Transactional
	public void internal() {
		...
	}
}
```

- **public 메서드**에만 트랜잭션이 적용 가능하다
```Java
@Transactional
public class Hello {
  public method1();
  method2():
  protected method3();
  private method4();

}
```
- public이 아닌 메서드에 적용한다고 해서 예외가 발생하는 것은 아니고 트랜잭션이 무시된다

#### @Transactioinal 옵션
- `value`, `transactionManager`
	- 트랜잭션 매니저의 스프링 빈 이름을 지정한다
	- 생략되면 기본으로 등록된 트랜잭션 매니저를 사용한다
- `rollbackFor`
	- 어떤 예외가 발생했을 때 롤백을 할 지 지정할 수 있다
	- `@Transactional(rollbackFor = Exception.class)`
	- `Exception`과 그 하위 예외들에 대해서 롤백할 수 있다
- `nonRollbackFor`
	- `rollbackFor`와 반대다
	- 어떤 예외가 발생했을 때 롤백하면 안되는지 지정할 수 있다
- `propagation`
	- 트랜잭션 전파에 관한 옵션이다
- `isolation`
	- 트랜잭션 격리 수준을 지정할 수 있다
- `timeout`
	- 트랜잭션 수행 시간에 대한 타임아웃을 초 단위로 지정한다
- `readOnly`
	- `readOnly=true`이면 읽기 전용 트랜잭션이 생성된다. 기본은 `false`이다
	- `readOnly`은 크게 3곳에 사용된다
	- 프레임워크
		- `jdbcTemplate`는 읽기 전용 트랜잭션 안에서 변경 기능을 실행하면 예외를 던진다
		- JPA는 읽기 전용 트랜잭션의 경우 커밋 시점에 플러시를 호출하지 않는다
		- 변경이 필요 없으니 스냅샷 객체도 생성하지 않는다
	- JDBC 드라이버
		- 읽기 전용 트랜잭션에서 변경 쿼리가 발생하면 예외를 던진다
		- 읽기, 쓰기 데이터베이스를 구분해서 요청한다
		- DB와 드라이버에 따라 달라질 수 있다
	- 데이터베이스
		- 데이터베이스에 따라 읽기 전용 트랜잭션의 경우 읽기만 하면 되므로 내부에서 성능 최적화가 발생한다

## 스프링 트랜잭션 전파
### 물리 트랜잭션과 논리 트랜잭션
- 스프링은 스프링에서 제공하는 트랜잭션 기능을 설명하기 위해 논리 트랜잭션과 물리 트랜잭션이라는 개념을 사용한다
- 논리 트랜잭션은 하나의 물리 트랜잭션으로 묶인다
- 물리 트랜잭션은 실제 데이터베이스에 적용되는 트랜잭션을 말한다. 실제 커넥션을 통해서 커밋, 롤백하는 단위다
- 논리 트랜잭션은 트랜잭션 매니저를 통해 트랜잭션을 사용하는 단위다
- **모든 논리 트랜잭션이 커밋되어야 물리 트랜잭션이 커밋된다**
- **하나의 논리 트랜잭션이라도 롤백되면 물리 트랜잭션은 롤백된다**

<img width="563" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/0cc39049-22c7-4644-b761-011d542d9228">
- 스프링은 기본적으로 외부 트랜잭션과 내부 트랜잭션이 모두 커밋되어야 물리 트랜잭션이 커밋된다
- 외부 트랜잭션은 여기서 처음으로 시작된 트랜잭션이고, 내부 트랜잭션은 외부 트랜잭션 수행 중 호출된 트랜잭션을 의미한다


### 트랜잭션 전파 - REQUIRES_NEW
- 물리 트랜잭션을 분리하려면 `REQUIRES_NEW` 옵션을 사용한다

<img width="564" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/8783c3f6-8ad7-4b2f-9ad1-1ae9f7a9e86b">

<img width="563" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/d9b683d2-7497-4046-a901-6c268f9fae47">
1. 외부 트랜잭션이 시작한다
2. 트랜잭션 매니저는 데이터소스를 통해 커넥션을 생성한다
3. 커넥션을 수동 커밋 모드로 설정한다. 물리 트랜잭션이 시작된다
4. 트랜잭션 매니저는 트랜잭션 동기화 매니저에 커넥션을 보관한다
5. 트랜잭션 매니저는 트랜잭션을 생성한 결과를 `TransactionStatus`에 담아서 반환하는데, 여기에 신규 트랜잭션 여부가 담겨있다. `isNewTransaction`을 통해 신규 트랜잭션 여부를 확인할 수 있다
6. 로직1이 사용되고, 커넥션이 필요한 경우 트랜잭션 동기화 매니저를 통해 트랜잭션이 적용된 커넥션을 획득해서 사용한다
7. `REQUIRES_NEW` 옵션과 함께 내부 트랜잭션이 시작한다. 새로운 트랜잭션을 시작한다
8. 트랜잭션 매니저는 데이터소스를 통해 커넥션을 생성한다
9. 생성한 커넥션을 수동 커밋 모드로 설정한다. 물리 트랜잭션이 시작된다
10. 트랜잭션 매니저는 트랜잭션 동기화 매니저에 커넥션을 보관한다. `con1`은 잠시 보류되고 `con2`가 사용된다
11. 트랜잭션 매니저는 신규 트랜잭션을 생성한 결과를 반환한다
12. 로직2가 사용되고, 커넥션이 필요한 경우 트랜잭션 동기화 매니저에 있는 `con2` 커넥션을 획득해서 사용한다

<img width="560" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/bfa50698-2ec7-49b2-b787-9cfedb4a92f1">

- 내부 트랜잭션이 새로운 물리 트랜잭션을 시작했기 때문에 내부 트랜잭션이 실패하는 경우 `con2` 물리 트랜잭션을 롤백한다
- 외부 트랜잭션은 `con1` 물리 트랜잭션을 커밋한다
- `REQUIRES_NEW`를 사용하면 데이터베이스 커넥션이 동시에 2개가 사용된다는 점을 주의해야 한다

### 다양한 트랜잭션 전파 옵션
- REQUIRED
	- 가장 많이 사용하는 전파 옵션
	- 기존 트랜잭션이 없으면 생성하고, 있으면 참여한다
- REQUIRE_NEW
	- 항상 새로운 트랜잭션을 생성한다
- SUPPORT
	- 기존 트랜잭션이 없으면 트랜잭션 없이 진행하고 있으면 참여한다
- NOT_SUPPORT
	- 트랜잭션을 지원하지 않는다
	- 기존 트랜잭션이 있어도 트랜잭션 없이 진행한다
- MANDATORY
	- 기존 트랜잭션이 없으면 예외가 발생한다
- NEVER
	- 기존 트랜잭션이 있으면 예외가 발생한다. 기존 트랜잭션도 허용하지 않는 강한 부정의 의미로 이해하자
- NESTED
	- 기존 트랜잭션이 없으면 새로운 트랜잭션을 생성한다
	- 기존 트랜잭션이 있으면 중첩 트랜잭션을 만든다
		- 중첩 트랜잭션은 외부 트랜잭션의 영향을 받지만, 중첩 트랜잭션은 외부에 영향을 주지는 않는다
		- 중첩 트랜잭션이 롤백 되어도 외부 트랜잭션은 커밋할 수 있다
		- 외부 트랜잭션이 롤백되면 중첩 트랜잭션도 함께 롤백된다