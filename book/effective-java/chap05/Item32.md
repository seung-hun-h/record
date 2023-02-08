## 제네릭과 가변인수를 함께 쓸 때는 신중하라
- 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 한다
- 하지만 가변인수는 배열을 클라이언트에 노출하는 문제가 있다
	- 그 결과 varargs 매개변수에 제네릭이나 매개 변수화타입이 포함되면 알기 어려운 컴파일 경고가 발생한다

- 메서드를 선언할 때 실체화 불가 타입으로 varargs 매개변수를 선언하면 컴파일러가 경고를 보낸다
	- 가변인수 메서드를 호출할 때도 varargs 매개변수가 실체화 불가 타입으로 추론되면, 그 호출에 대해서도 경고를 보낸다
```Text
warning: [unchecked] Possible heap pollution from
	parameterized vararg type List<String>
```

- 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다
	- 이러한 경우 제네릭이 제공하고자 했던 컴파일 타임에 타입 안전성을 해칠 수 있다

```Java
static void dangerous(List<String>... stringLists) {  
   List<Integer> intList = List.of(42);  
   Object[] objects = stringLists;  
   objects[0] = intList;  
   String s = stringLists[0].get(0);  
}
```

-  위 메서드를 실행하면 ClassCastException이 발생한다
- 이처럼 타입 안정성이 깨지니 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다

### 제네릭과 가변인수를 함께 사용하는 이유
- 그럼에도 제네릭과 가변인수를 함께 사용하는 이유는 유용하기 때문이다
	- `Arrays.asList(T... a)`, `Collections.addAll(Collection<? super T> c, T... elements)`, `EnumSet.of(E first, E... rest)` 등이 있다
	- 이들은 타입 안전하다

### @SafeVarargs
- `@SafeVarargs` 애너테이션은 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치이다
	- 메서드가 안전한 것이 확실치 않다면 달면 안된다
- **varargs 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수를 전달하는 일만 한다**면 그 메서드는 안전하다
	- 메서드가 이 배열에 아무것도 저장하지 않고
	- 그 배열의 참조가 외부로 노출되지 않는다면 안전하다

#### 예외 경우
- varargs 매개변수 배열에 아무것도 저장하지 않고도 타입 안전성이 깨질 수 있다
```Java
static <T> T[] toArray(T... args) {
	return args;
}
```
- 이 메서드가 반환하는 배열의 타입은 컴파일 타임에 결정되는데, 컴파일 시점에는 컴파일러에게 충분한 정보가 주어지지 않아 타입을 잘못 판단할 수 있다

**예시**

```Java
static <T> T[] pickTwo(T a, T b, T c) {
	switch(ThreadLocalRandom.current().nextInt(3)) {
		case 0: return toArray(a, b);
		case 1: return toArray(a, c);
		case 2: return toArray(b, c);
	}
	throw new AssertionError();
}
```

```Java
public static void main(String[] args) {
	String[] attributes = pickTwo("좋은", "빠른", "저렴한");
}
```

- 위 코드는 ClassCastException을 던진다
- 제네릭은 실체화 불가하므로 `pickTwo`는 `Object[]`를 반환한다
- 하지만 `String[]`는 `Object[]` 의 하위타입이 아니므로 형변환은 실패한다
- 이는 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다는 점을 다시 상기시킨다
	- 예외1: `@SafeVarargs`가 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 것은 안전하다
	- 예외2: 이 배열 내용의 일부 함수를 호출만 하는 (varargs)일반 메서드에 넘기는 것도 안전하다

#### 제네릭 가변인수 배열에 다른 메서드가 접근해도 되는 경우
1. `@SafeVarargs`가 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 것은 안전하다

```Java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
	List<T> result = new ArrayList<>();
	for (List<? extends T> list : lists) {
		result.addAll(list);
	}
	return result;
}
```
- 위 메서드는 선언하는 쪽과 사용하는 쪽 모두 경고를 내지 않는다
- 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 `@SafeVarargs`를 달아라
	- 이 말은 안전하지 않은 varargs 메서드는 절대 작성하면 안된다는 뜻이다
- 통제할 수 있는 메서드 중 varargs 매개변수를 사용하며 힙 오염 경고가 뜨는 메서드가 있으면, 그 메서드가 진짜 안전한지 점검해라
- 둘 중 하나라도 어기면 수정해라
	- varargs 매개변수 배열에 아무것도 저장하지 않는다
	- 그 배열을 신뢰할 수 없는 코드에 노출하지 않는다

2. varargs 매개변수를 List 매개변수로 바꾼다
```Java
@SafeVarargs
static <T> List<T> flatten(List<List<? extends T>> lists) {
	List<T> result = new ArrayList<>();
	for (List<? extends T> list : lists) {
		result.addAll(list);
	}
	return result;
}
```

```Java
audience = flatten(List.of(friends, romans, countrymen));
```
- 정적 팩터리 메서드인 `List.of`를 사용하면 메서드에 임의 개수의 인수를 넘길 수 있다

```Java
static <T> List<> pickTwo(T a, T b, T c) {
	switch(ThreadLocalRandom.current().nextInt(3)) {
		case 0: return List.of(a, b);
		case 1: return List.of(a, c);
		case 2: return List.of(b, c);
	}
	throw new AssertionError();
}
```

```Java
public static void main(String[] args) {
	List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
}
```

- `List.of`를 사용하면 위 같이 타입안전한 코드를 간편하게 작성할 수 있다
- 배열없이 제네릭만 사용하므로 타입 안전하다