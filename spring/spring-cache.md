# Spring Cache Abstraction
- 스프링은 AOP 방식으로 편리하게 메소드에 캐시를 적용할 수 있도록 추상화를 제공한다
- 스프링은 아래와 같은 몇가지 캐시 구현체를 제공한다
  - JDK `ConcurrentHashMap` based Cache
  - Ehcache-based Cache
  - Gemfire-based Cache
  - Caffeine Cache
  - JSR-107 Cache
- `org.springframework.cache.Cache`, `org.springframework.cache.CacheManager` 가 스프링 캐시 추상화의 핵심적인 역할을 한다
  - `CacheManager` 인스턴스는 하부 캐시 솔루션이 제공하는 캐시 매니저를 감싸는 래퍼 역할을 하며, `Cache` 인스턴스를 관리한다
  - `Cache` 인스턴스는 하부 캐시를 감싸며, 하부 캐시와 상호 작용하는 메서드를 제공한다

## Spring 캐시 사용하기

### 1. @EnableCaching 추가
```java
@EnableCaching
@SpringBootApplication
public class SpringCacheApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCacheApplication.class, args);
	}
}
```

### 2. 캐시 매니저 빈 등록
- ConcurrentMapCacheManager: Java의ConcurrentHashMap을 사용해 구현한 캐시를 사용하는 캐시매니저
- SimpleCacheManager: 기본적으로 제공하는 캐시가 없어 사용할 캐시를 직접 등록하여 사용하기 위한 캐시매니저
- EhCacheCacheManager: 자바에서 유명한 캐시 프레임워크 중 하나인 EhCache를 지원하는 캐시 매니저
- CompositeCacheManager: 1개 이상의 캐시 매니저를 사용하도록 지원해주는 혼합 캐시 매니저
- CaffeineCacheManager: Java 8로 Guava 캐시를 재작성한 Caffeine 캐시를 사용하는 캐시 매니저
- JCacheCacheManager: JSR-107 기반의 캐시를 사용하는 캐시 매니저

### 3. 캐시 어노테이션 사용
**@Cacheable**
- 캐시에 데이터가 없을 경우에는 기존의 로직을 실행한 후, 캐시에 데이터를 추가하고, 캐시에 데이터가 있으면 캐시에 저장된 데이터를 반환한다
- `key` 속성은 반환한 값을 캐시에 저장할 때 사용할 키를 의미한다
  - `key`를 지정하지 않으면 기본적으로 스프링의 `SimpleKeyGenerator` 클래스를 사용한다
  - `SimpleKeyGenerator`는 메서드 시그니처와 인수를 사용해 키를 생성한다
    - 메서드에 전달한 인수가 모두 같을때는 `@Cacheable`을 설정한 메서드를 호출하지 않는다
    - 인자가 없을 경우에는 기본값인 0을 key로 하여 저장한다
    - 인자가 여러개인 경우 인자의 hashCode값을 조합하여 key로 설정한다
- `cacheNames` 속성은 반환한 값을 캐시할 캐시 영역을 지정한다
- `condition` 속성은 특정 조건일 때만 캐시를 적용할 때 사용한다

**@CacheEvict**
- 캐시에 있는 데이터를 비울 때 사용한다
- `cacheNames`는 비울 캐시의 영역을 의미한다
- `condition`은 조건에 따라 캐시를 비울 때 사용한다
- `allEntries`는 지정한 캐시 영역의 모든 엔트리를 비울 지 일부만 비울지 지정한다
- `beforeInvocation`은 캐시를 메서드 실행 이전이나 이후 중 언제 비울지 결정한다

**@CachePut**
- 항상 메서드를 호출해서 반환 값을 넣을 때 사용한다
  - `Cacheable`과 다르게 항상 메서드가 호출된다