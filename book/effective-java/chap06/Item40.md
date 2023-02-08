## @Override 애너테이션을 일관되게 사용하라
---
```Java
public class Bigram {

	...
	public boolean equals(Bigram b) {
		return b.first == first && b.second == b.second;
	}
}
```
- `equals(Bigram)`은 `Object`의 `equals`를 재정의한 것이 아니라 다중정의한 것이다
	- `Object`는`equals(Object)`처럼 Object 타입을 인자로 받아야 한다
- `@Override` 애노테이션을 재정의하지 않은 메서드에 사용하면 컴파일 에러가 발생한다
- 따라서 상위 클래스의 메서드를 재정의하려는 모든 메서드에 `@Override` 애너테이션을 사용해야 한다
	- 단, 구체 클래스에서 상위 클래스의 추상 메서드를 구현할 때는 굳이 달지 않아도 된다
	- 아직 구현하지 않은 메서드가 있다면 컴파일러가 알려주기 때문이다
- 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스를 재정의하는 모든 메서드에 `@Override`를 다는 것이 좋다
	- 시그니처가 올바른지 재차 확인할 수 있다