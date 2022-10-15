## ITEM21 인터페이스 타입을 정의하는 용도로만 사용하라
---
- 인터페이스는 자신을 구현한 클래스의 인터페이스를 참조할 수 있는 타입 역할을 한다
	- 인터페이스는 오직 이 용도로 사용해야 한다

### 상수 인터페이스
- 상수들을 모아 놓은 상수 인터페이스는 이 지침에 맞지 않는 대표적인 예이다
- 그리고 이 상수들을 사용하려는 클래스에서는 정규화된 이름을 쓰는 걸 피하고자 그 인터페이스를 구현하곤한다(?)

```Java
public interface PhysicalConstants {
	static final double AVOGARDROS_NUMBER = ...;
	static final double BOLTZMANN_CONSTANT = ...;
	static final double ELECTRON_MASS = ...;
}
```

- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다
- 상수 인터페이스를 구현하는 것은 내부 구현을 클래스의 API로 노출하는 행위다
	- 클래스가 어떤 상수 인터페이스를 사용하든 사용자에게는 의미없다
	- 더 심하게는 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 의존하게된다
- 다음 릴리즈에서 이 상수들을 더이상 쓰지 않게 되더라도 바이너리 호환성을 위해 여전히 상수 인터페이스를 구현해야 한다(?)

### 상수 공개 방법
- 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야 한다
- 열거타입으로 나타내기 적합한 상수라면 열거 타입으로 공개하면된다
- 그것도 아니라면 인스턴스화 할 수 없는 유틸리티 클래스에 담아 공개한다

```Java
public class PhysicalConstants {
	private PhysicalConstants() { ... }

	public static final double AVOGARDROS_NUMBER = ...;
	public static final double BOLTZMANN_CONSTANT = ...;
	public static final double ELECTRON_MASS = ...;
}
```

