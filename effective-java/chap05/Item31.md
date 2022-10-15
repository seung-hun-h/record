## ITEM31 한정적 와일드카드를 사용해 API 유연성을 높이라

- 유연성을 극대화 하기 위해서는 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라
- `Stack`의 public API을 추리면 아래와 같다
```Java
public class Stack<E> {
	public Stack();
	public void push(E e);
	public E pop();
	public boolean isEmpty();
}
```
- 여기에 일련의 원소들을 스택에 추가하는 메서드를 구현해야 한다고 가정한다
```Java
publi void pushAll(Iterable<E> src) {
	for (E e : src) {
		push(e);
	}
}
```

- 위 코드는 컴파일 되지만 완벽하지 않다
	- `Stack<Number>`를 선언해도 `Iterable<Integer>`를 매개변수로 전달할 수 없기 때문이다.
	- 제네릭은 불공변이다
- 이 문제는 한정적 와일드카드를 사용하면 해결할 수 있다

```Java
publi void pushAll(Iterable<? extends E> src) {
	for (E e : src) {
		push(e);
	}
}
```

- `pushAll`과 짝을 이루는 `popAll`을 구현하면 아래와 같을 것이다

```Java
public void popAll(Collection<E> dst) {
	while(!isEmpty()){
		dst.add(pop());
	}
}
```
- 이 코드도 컴파일은 되지만 잠재적으로 문제를 가지고있다
	- `Collection<Object>`는 `Collection<Integer>`의 상위 타입이 아니기 때문이다
- 이 문제도 한정적 와일드카드로 해결할 수 있다

```Java
public void popAll(Collection<? super E> dst) {
	while(!isEmpty()){
		dst.add(pop());
	}
}
```

### PECS: producer-extends, consumer-super
- 제네릭이 사용되는 곳이 생성자면 extends를 사용하고, 소비자면 super를 사용한다
- `pushAll`의 `Iterable`은 원소를 제공하는 생성자이므로 extends를 사용하고, `popAll`은 원소를 사용하는 소비자이므로 super를 사용했다
- 다음은 일련의 원소 중 최대값을 구하는 함수인 max이다
```Java
public static <E extends Comparable<E>> E max(List<E> list);
```
- 이를 한정적 와일드카드로 다듬으면 아래와 같다

```Java
public static <E extends Comparable<? super E>> E max(List<? extends E> list);
```

- 한정적 와일드카드를 사용해 정의가 난해해지긴 했지만 첫번째보다 낫다
- 원래 선언에서는 E가 `Comparable<E>`를 확장한다고 정의했는데, 이떄 `Comarable<E>`는 E 인스턴스를 소비한다
	- 그래서 매개변수화 타입 `Comparable<E>`를 한정적 와일드카드 타입인 `Comparable<? extends E>`로 대체했다
	- 일반적으로 `Comparable`은 소비자이므로 super를 사용한다
- 굳이 이렇게 난해하기 정의해야하나 싶지만, 확실히 더 낫다
	- `List<ScheduledFuture<?>> scheduledFutures = ...;`는 수정전의 `max`로는 처리할 수 없다
	- `ScheduledFuture`는 `Comparable<ScheduledFuture>`를 구현하지 않았다
	- `ScheduledFuture`는 `Delayed`의 하위타입이고, `Delayed`는 `Comparable<Delayed>`를 구현했다
	- `ScheduledFuture`는 다른 `ScheduledFuture` 인스턴스뿐 아니라 `Delayed` 인스턴스와도 비교할 수 있어서 수정전 `max`는 사용할 수 없다
	- 일반화해서 말하면 `Comparable`을 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 와일드카드가 필요하다

### 타입 매개변수와 와일드카드 중 어떤것을 사용할까?
- 타입 매개변수와 와일드카드에 공통되는 부분이 있어서, 메서드를 정의할 때 둘 중 어느것을 사용해도 괜찮을 때가 많다
- 예를 들어 주어진 리스트에서 명시한 두 인덱스의 아이템들을 교환하는 정적 메서드를 두 가지 방식으로 정의해본다

```Java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

- public API라면 둘 중 일반적으로 두 번째가 낫다
- 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하는 것이 좋다
	- 비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면된다
- 하지만 아래와 같이 구현한 코드는 컴파일되지 않는다

```Java
public static void swap(List<?> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```
- `List<?>`에는 null외에 어떤 값도 넣을 수 없기 때문에 문제가 발생한다
- 이때 형변환이나 리스트의 로 타입을 사용하지 않고, 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 따로 작성하여 활용하면 해결할 수 있다

```Java
public static void swap(List<?> list, int i, int j) {
	swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```
- 내부에서는 더 복잡한 제네릭 메서드를 사용했지만, 외부에서는 와일드카드 기반의 멋진 선언을 유지할 수 있었다
	- 여기에서는 간단한 선언을 위해서 타입 매개변수가 아닌 와일드카드를 사용한 것일까?(?)