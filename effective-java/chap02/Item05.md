## ITEM05 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

>
>클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 이 자원들을 클래스가 직접 만들게 해서도 안된다. 대신 필요한 자원을 생성자에 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해준다
>

- 맞춤법 검사기가 있다고 가정했을때, 맞춤법 검사기는 사전에 의존한다
- 이떄 맞춤법 검사기를 정적 유틸 클래스나 싱글턴으로 구현하는 경우가 있다

```Java
public class SpellChecker {
	private static final Lexion dictionary = ...;

	private SpellChecker() { ... }

	public static boolean isValid(String word) {...}
}
```

```Java
public class SpellChecker {
	private static final Lexion dictionary = ...;
	public static SpellChecker INSTANCE = new SpellChecker(...);

	private SpellChecker(...) { ... }

	public static boolean isValid(String word) {...}
}
```

- 두 방식은 모두 단 한 종류의 사전을 사용한다고 가정한다
- 사전은 언어별로 따로 존재할 수 있고, 특수 어휘용 사전도 있을 수 있다
- final을 삭제하고 다른 사전으로 교체하는 메서드를 추가할 수 있을 것이다. 하지만 이러한 방법은 멀티스레드 환경에서는 적합하지 않다
- 이러한 문제점을 해결하기 위해 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식을 사용할 수 있다

### 의존성 주입
- 한 클래스가 의존하는 다른 클래스의 인스턴스를 외부에서 주입해주는 것을 의존성 주입이라 한다

```Java
public class SpellChecker {
	private final Lexion dictionary;

	public SpellChecker(Lexion dictionary) {
		this.dictionary = dictionary;
	}

	...
}
```
- 생성자에 의존성을 주입해주면 final을 사용할 수 있어 불변을 보장해 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있다
- 이 패턴의 변형으로 생성자에 자원 팩토리를 넘겨주는 방식이 있다
- 자바의 `Supplier<T>`는 팩터리를 표현한 완벽한 예이다
