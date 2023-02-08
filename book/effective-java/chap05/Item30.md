## ITEM30 이왕이면 제네릭 메서드로 만들라

- 클래스와 마찬가지로 메서드도 제네릭으로 만들 수 있다
- 메서드에 제네릭을 사용하면 타입 매개변수 목록을 작성해야 한다
	- 타입 매개변수 목록은 메서드 제한자와 반환 타입 사이에 온다

```Java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
	Set<String> result = new HashSet<>(s1);
	result.addAll(s2);
	return result;
}
```

- 한정적 와일드카드 타입을 사용하면 위의 메서드를 더욱 유연하게 만들 수도 있다

### 제네릭 싱글턴 팩터리
- 제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다
- 제네릭 싱글턴 팩터리는 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 말한다
	- `Collections.reverseOrder`, `Collections.emptSet` 등등이 존재한다

#### 항등 함수

```Java
private static UnaryOperator<Object> IDENTIFY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identifyFunction() {
	return (UnaryOperator<T>)IDENTIFY_FN;
}
```
- `identifyFunction()`은 제네릭 싱글턴 팩터리 패턴을 사용한 예이다
- `IDENTIFY_FN`을 `UnaryOperator<T>`로 형변환하면 비검사 형변환 경고가 발생한다
- 하지만 항등 함수란 입력 값을 수정 없이 그대로 반환하는 특별한 함수이므로, `T`가 어떤 타입이든 `UnaryOperator<T>`를 사용해도 타입 안전하다
	- `@SuppressWarnings`를 사용해 비검사 형변환 경고를 제거한다

### 재귀적 타입한정
- 재귀적 타입한정이란 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정하는 것을 말한다
- 재귀적 타입한정은 주로 `Comparable` 인터페이스와 함께 쓰인다
```Java
public interface Comparable<T> {
	int compareTo(T o);
}
```
- 타입 매개변수 `T` 는 `Comparable<T>`를 구현한 타입이 비교할 수 있는 원소 타입을 정의한다
	- `String`은 `Comparble<T>`, `Integer`는 `Comparable<Integer>`를 구현한다
- 원소의 순서를 정하거나 최대, 최소를 구하려면 모든 원소가 상호 비교될 수 있어야 한다. 재귀적 한정 타입을 사용하면 아래와 같이 작성할 수 있다

```Java
public static <E extends Comparable<E>> E max(Collection<E> c);
```

- `<E extends Comparable<E>>`는 '모든 타입 E는 자신과 비교할 수 있다'로 해석할 수 있다

---
- https://mangkyu.tistory.com/241