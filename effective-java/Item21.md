## ITEM21 인터페이스는 구현하는 쪽을 생각해 설계하라
---
- 자바 8에서 기존 인터페이스에 디폴트 메서드를 추가할 수 있도록 했지만 위험이 완전히 사라진 것은 아니다

### 불변식을 해치지 않는 디폴트 메서드를 작성하기는 어렵다
- 자바 8의 Collection에는 `removeIf()`가 추가되었다
- removeIf는 프레디케이트를 인자로 받아 true를 반환하는 경우 원소를 제거한다

```Java
default boolean removeIf(Predicate<? super E> filter) {
	Objects.requireNonNull(filter);
	boolean result = false;

	for (Iterator<E> it = iterator(); it.hasNext();) {
		if (filter.test(it.next())) {
			it.remove();
			result = true;
		}
	}

	return result;
}
```

- 잘 구현된 메서드이지만, 현존하는 모든 Collection 구현체와 잘 어우러지는 것은 아니다
	- `org.apache.commons.collections4.collection.SynchronizedCollection`이 그 예이다
	- 이 클래스는 `java.util`의 `Collections.synchronizedCollection` 정적 팩터리 메서드가 반환하는 클래스와 비슷하다
	- 모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스다
- 아파치의 SynchronizedCollection 클래스는 이 책이 쓰여진 시점엔 removeIf를 재정의 하지 않고 있다
	- 따라서 모든 메서드 호출을 알아서 동기화해주지 못한다
	- 여러 스레드가 공유하는 환경에서 한 스레드가 removeIf를 호출하면 ConcurrentModificationException이 발생하거나 치명적인 문제가 발생한다
	- 4.4 버전 소스코드인데 이건 재정의 한건가?(?)

```Java
public boolean removeIf(Predicate<? super E> filter) {  
    synchronized(this.lock) {  
        return this.decorated().removeIf(filter);    
	}  
}
```

#### 자바 플랫폼 라이브러리에서의 조치
- 이러한 문제를 해결하기 위해서 자바 플랫폼 라이브러리에서 여러 조취를 취했다
- 구현한 인터페이스의 디폴트 메서드를 재정의하고, 다른 메서드에서는 디폴트 메서드를 호출하기 전에 필요한 작업을 수행하도록 했다
	- `Collections.synchronizedCollection`이 반환하는 package-private 클래스들은 removeIf를 재정의하고 이를 호출하는 다른 메서드들은 디폴트 구현을 호출하기 전에 동기화 하도록했다
	- 하지만 자바 플랫폼에 속하지 않은 제 3의 기존 컬렉션 구현체들은 이런 언어 차원이 인터페이스 변화에 발맞춰 수정될 기회가 없었고 그중 일부는 여전히 수정되지 않고 있다

---

### 디폴트 메서드는 기존 구현체에 런타임 오류를 일으킬 수 있다
- 흔하지는 않지만 일어나지 않으리라는 보장은 없다
- 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니라면 피해야 한다
	- 추가하려는 디폴트 메서드가 기존 구현과 충돌하지 않는지도 확인해야 한다
- 디폴트 메서드는 인터페이스로부터 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도가 아님을 명심해야 한다

---

### 마무리
- 디폴트 메서드라는 도구가 생겼어도 인터페이스를 설계할 때는 세심한 주의를 기울여야 한다
- 기존 인터페이스에 디폴트 메서드를 추가하면 커다란 위험도 같이 따라온다
- 새로운 인터페이스라면 릴리즈 전에 반드시 테스트 해봐야 한다
- 인터페이스를 릴리즈한 후라도 결함을 수정하는게 가능할 수 있지만, 절대 그 가능성에 기대서는 안된다
