## 옵셔널 반환은 신중히 하라
---
- 자바 8 이전에는 메서드가 값을 정상적으로 반환할 수 없을 때는 2가지 선택지가 있었다
	- 예외를 던진다
	- null을 반환한다
- 하지만 각각의 방법은 두 가지 문제점을 지니고 있다
	- 예외는 항상 예외적인 상황에서만 던져야 한다
	- null을 반환하면 클라이언트에서 null 방어 코드를 작성해야 한다
- 자바 8에서는 `Optional`을 제공해 두 가지 방법의 문제점을 해결하고자 했다

```Java
public static <E extends Comparable<E>> E max(Collection<E> c) {
	if (c.isEmpty()) {
		throw new IllegalArgumentException("빈 컬렉션");
	}

	E result = null;
	for (E e : c) {
		if (result == null || e.compareTo(result) > 0) {
			result = Objects.requireNonNull(e);
		}
	}
	return result;
}
```

- 위 코드에 `Optional`을 사용하면 아래처럼 작성할 수 있다

```Java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
	if (c.isEmpty()) {
		return Optional.empty();
	}

	E result = null;
	for (E e : c) {
		if (result == null || e.compareTo(result) > 0) {
			result = Objects.requireNonNull(e);
		}
	}
	return Optional.of(result);
}
```

- `Optional.emty()`는 빈 옵셔널을 생성한다
- `Optional.of(value)`는 값이 든 옵셔널을 생성한다
	- 하지만 value에 null을 넣으면 `NullPointerException`이 발생하니 주의해야 한다
- `Optional.ofNullabe(value)`을 사용하면 null 값을 허용하는 옵셔널을 만들 수 있다
- 옵셔널을 반환하는 메서드에서 null을 반환하지 말자

### 메서드가 옵셔널을 반환하는 경우
- 옵셔널은 검사 예외와 취지가 비슷하다
	- 반환값이 없을 수도 있음을 API 사용하자에게 명확히 알려준다
- 메서드가 옵셔널을 반환하면 클라이언트는 값을 받지 못했을 때 취할 행동을 선택해야 한다

- `orElse`를 사용해 기본값을 설정할 수 있다
```Java
String lastWordInLexicon = max(words).orElse("단어 없음...")
```
- `orElseThrow` 를 사용해 상황에 맞는 예외를 던질 수도 있다
```Java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```
- 옵셔널이 항상 값을 가지고 있다고 확신하면 `get()`으로 곧바로 값을 꺼내 사용할 수 있다
	- 단, 값이 없을 때는 `NoSuchElementException`이 발생하니 주의해야 한다
```Java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```
- 기본값을 설정하는 비용이 아주 커서 부담될 때는 `Supplier<T>`를 인자로 받는 `orElseGet()`을 사용하면 된다
- 더 특별한 쓰임새로 제공하는 메서드는 `filter`, `map`, `flatMap`, `ifPresent`가 있다
	- 기본으로 제공하는 메서드로 처리하기 힘들다면 이 고급 메서드를 사용하면 된다

### 옵셔널을 반드시 사용해야 하는 것은 아니다
- 반환값으로 옵셔널을 사용하는 것이 반드시 득이되는 것은 아니다
- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다
	- 그냥 빈 값을 반환하는 편이 좋다
- 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야하면 `Optional<T>`를 반환하도록 하자

### 박싱된 기본 타입을 담은 옵셔널은 반환하지 말자
- 박싱된 기본 타입을 감싼 옵셔널을 반환하는 것은 무겁다
	- 기본 타입을 감싼 박싱된 타입에 다시 옵셔널을 감싸기 때문에 두 번 감싸는 꼴이다
- int, long, double 전용 옵셔널 클래스가 있다
	- OptionalInt, OptionalLong, OptionalDouble
- 덜 중요한 기본타입인 Boolean, Byte, Character, Short, Float은 예외다
-

### 옵셔널을 사용하면 안되는 경우
- 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하지 말자
- 맵의 경우 키에 대한 값으로 옵셔널을 사용하면 키가 가진 상태가 두 가지가된다
	- 키 자체가 맵에 없는 경우
	- 키에 매핑된 값이 빈 옵셔널인 경우
- 옵셔널을 인스턴스 필드에 저장해두는 경우는 '나쁜 냄새'에 해당한다
	- 필수 필드를 갖는 클래스와 이를 확장해 선택적 필드를 추가한 하위 클래스를 따로 만들어야 함을 암시한다
	- 게터에서 옵셔널을 반환할 수도 있다
