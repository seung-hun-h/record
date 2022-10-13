## ITEM27 비검사 경고를 제거하라
- 비검사 경고(Unchecked warning)
	- 비검사 형번환 경고
	- 비검사 메서드 호출 경고
	- 비검사 매개변수화 가변인수 타입 경고
	- 비검사 변환 경고

> The term "unchecked" refers to the fact that the compiler and the runtime system do not have enough type information to perform all type checks that would be necessary to ensure type safety.
> 
> - https://stackoverflow.com/questions/1129795/what-is-suppresswarnings-unchecked-in-java

- 할 수 있는 한 모든 비검사 경고를 제거해야 한다
	- 비검사 경고를 제거할 수 있으면, 그 코드는 타입 안정성을 보장한다

### @SuppressWarnings
- 경고를 제거할 수 없지만 타입 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchecked")` 어노테이션을 달아 경고를 숨기자
	- 타입이 안전하다고 보장되지 않은 채 사용하면 스스로에게 잘못된 보안 인식을 심어주는 꼴이다
	- 안전하다고 검증된 코드에 비검사 경고를 제거하지 않으면 수 많은 거짓 경고 속에 새로운 경고가 무시당할 수 있다
- `@SuppressWarnings`애너테이션은 항상 가능한 한 좁은 범위에 적용하자

```Java
public <T> T[] toArray(T[] a) {
	if (a.length < size) {
		return (T[]) Arrays.copyOf(elements, size, a.getClass());
	}
	System.arraycopy(elements, 0, a, 0, size);
	if (a.length > size) {
		a[size] = null;
	}
	return a;
}
```
- 자바 ArrayList의 toArray 메서드를 예로 들었다. 위 코드는 비검사 경고가 발생한다.
	- 비검사 경고를 해결하기 위해 `@SuppressWarnings` 애너테이션을 사용하되 최대한 좁은 범위에서 사용해야 한다.
	- 위 코드에서는 return 구문에는 애너테이션을 사용하는 것이 불가능하므로 지역변수에 할당하는 방식을 사용한다

```Java
public <T> T[] toArray(T[] a) {
	if (a.length < size) {
		@SuppressWarnings("unchecked") T[] result = 
					(T[]) Arrays.copyOf(elements, size, a.getClass());
		return result;
	}
	System.arraycopy(elements, 0, a, 0, size);
	if (a.length > size) {
		a[size] = null;
	}
	return a;
}
```

- `@SuppressWarnings` 애너테이션을 사용할 떄는 항상 그 경고를 무시해도 되는 이유를 주석으로 달아야 한다

