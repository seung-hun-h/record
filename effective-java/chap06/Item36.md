## 비트 필드 대신 EnumSet을 사용하라
---
- 열거한 값들이 집합으로 사용되는 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔다

```Java
public class Text {
	public static final int STYLE_BOLD = 1 << 0;
	public static final int STYLE_ITALIC = 1 << 1;
	public static final int STYLE_UNDERLINE = 1 << 2;
	public static final int STYLE_STRIKETHROUGH = 1 << 3;

	public void applyStyles(int styles) { ... }
}
```

- 다음과 같은 식으로 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모으 수 있으며, 이렇게 만들어진 집합을 비트 필드라고 한다

```Java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```
- 비트 필드의 장점
	- 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다
- 비트 필드의 단점
	- 정수 열거 패턴의 단점을 그대로 가지고있다
	- 비트 필드 값이 그대로 출력되면 해석하기 어렵다
	- 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 어렵다
	- 최대 몇 비트가 필요한지를 API 작성시 미리 예측하여 적절한 타입을 선택해야 한다

### EnumSet을 사용하자
- `EnumSet`은 열거 타입 상수의 값으로 구성된 집합을 효율적으로 표현해준다
- `Set`인터페이스를 완벽히 구현해 다른 구현체와 같이 사용될 수 있다
- `EnumSet`의 내부는 비트 벡터로 구현되어 원소가 총 64개 이하라면 비트 필드에 비견되는 성능을 보여준다
	- 성능은 비트 필드 만큼, 비트를 직접 다룰때 난해함을 해결해주었다

```Java
public class Text {
	public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

	public void applyStyle(Set<Style> styles) { ... }
}
```

- `applyStyles`메서드가 `EnumSet<Style>`이 아닌 `Set<Style>`을 받은 이유는 범용성 때문이다
	- 모든 클라이언트가 `EnumSet`을 전달하리라 짐작되는 상황에서도 인터페이스로 받는 것이 좋은 습관이다
