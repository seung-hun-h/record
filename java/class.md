# 클래스
## 객체지향 프로그래밍의 특징
객체지향 프로그래밍(OOP)란 프로그램을 여러 객체로 나누고 객체간 메시지 교환을 통해서 프로그램이 동작하는 프로그래밍 방식을 의미한다.

객체지향 프로그래밍의 특징은 추상호, 캡슐화, 다형성, 상속 네 가지가 있다.

- 추상화: 객체에서 공통된 속성이나 기능을 추출하는 것이다. 중요하지 않는 것은 무시하고 중요한 것만 강조하여 추출한다.
- 캡슐화: 객체의 필드, 메소드를 하나로 묶고 실제 구현 내용을 외부로부터 감추는 것을 의미한다. 캡슐화를 통해서 외부에서 마음대로 객체의 값을 변경시키는 것을 막을 수 있다. 자바는 접근 제한자를 통해서 캡슐화가 가능하다.
- 다형성: 같은 타입이지만 실행 결과가 다양한 객체를 사용할 수 있는 성질을 말한다. 부모 클래스나 인터페이스를 상속하거나 구현한 자식 클래스가 부모 타입으로 변환하는 것을 허락하여 타입은 동일하지만 다양한 기능을 제공할 수 있다.
- 상속: 상위 객체가 가지고 있는 필드와 메소드를 하위 객체에게 물려주어 하위 객체가 사용할 수 있도록한다. 상위 객체를 재사용하여 코드의 중복을 줄여준다.

### Overriding VS Overloading

- 오버라이딩: 상위 클래스 혹은 인터페이스에 존재하는 메소드를 하위 클래스에서 필요에 맞게 재정의 하는 것을 의미한다
- 오버로딩: 클래스 내에 같은 이름의 메소드를 여러 개 선언하는 것을 의미한다. 매개 변수의 타입, 개수, 순서 중 하나가 달라야하며, return 타입을 동일하거나 다를 수 있다.

### 인스턴스 멤버와 this
인스턴스는 객체를 생성한 후 사용할 수 있는 필드와 메소드를 말한다. 인스턴스 필드, 인스턴스 메소드라고 부른다. 객체 내부에서 인스턴스 멤버에 접근하기 위해 this를 사용할 수 있다. 주로 인스턴스 필드 이름과 메서드의 매개변수 이름이 동일할 때 인스턴스 필드를 명시할 때 this를 사용한다.

### 정적 멤버와 static
정적 멤버는 클래스에 고정된 멤버로 객체를 생성하지 않고 사용할수 있는 필드와 메서드를 말한다. 각각 정적 필드, 정적 메서드라고 하며 클래스 멤버라고도 한다.

정적 멤버를 선언하기 위해서는 static 키워드를 추가해야한다. 정적 필드와 정적 메서드는 클래스 로더가 클래스를 로딩해서 메소드 메모리 영역에 적재할 때 클래스 별로 관리한다. 따라서 클래서 로딩이 끝나면 바로 사용할 수 있다.

**정적 초기화 블록**

인스턴스 멤버는 생성자에서 초기화 작업을 할 수 있지만, 정적 멤버는 생성자를 사용할 수 없다 따라서 초기화 하는 방법이 필요한데, 정적 멤버의 복잡한 초기화 작업을 위해 자바는 정적 블록을 제공한다. 정적 블록은 클래스가 메모리로 로딩될 때 자동적으로 실행된다. 정적 블록은 클래스 내부에 여러개가 선언되어도 상관없다.

## 싱글톤
싱글톤은 클래스의 동일한 인스턴스를 사용할 수 있게하는 디자인 패턴이다. 싱글톤 인스턴스의 생성 방법은 다양하다.

### Eager initialization

```java
public class EagerInitialization {
	// private static 로 선언.
	private static EagerInitialization instance = new EagerInitialization();
	// 생성자
	private EagerInitialization () {
		System.out.println( "call EagerInitialization constructor." );
	}
	// 조회 method
	public static EagerInitialization getInstance () {
		return instance;
	}
	
	public void print () {
		System.out.println("It's print() method in EagerInitialization instance.");
		System.out.println("instance hashCode > " + instance.hashCode());
	}
}
```
`private static ... = new ..()`처럼 클래스가 로딩된 직후 바로 인스턴스를 생성한다. 프로그램의 크기가 커질 경우 사용하지도 않는 인스턴스가 생성되는 것이 부담될 수 있고, 인스턴스화 되는 중간에 어떠한 작업도 할 수 없다는 단점이 있다.

### static block initialization
```java
public class StaticBlockInitalization {
	private static StaticBlockInitalization instance;
	private StaticBlockInitalization () {}
	
	static {
		try {
			System.out.println("instance create..");
			instance = new StaticBlockInitalization();
		} catch (Exception e) {
			throw new RuntimeException("Exception creating StaticBlockInitalization instance.");
		}
	}
	
	public static StaticBlockInitalization getInstance () {
		return instance;
	}
	
	public void print () {
		System.out.println("It's print() method in StaticBlockInitalization instance.");
		System.out.println("instance hashCode > " + instance.hashCode());
	}
	
}
```
정적 초기화 블럭을 사용하면 클래스가 로딩 될 때 최초 한번 실행된다. 초기화 블럭을 사용하여 로직을 담을 수 있지만, 클래스가 로딩되는 직후 인스턴스가 생성된다는 단점은 여전하다.

### lazy initializtion
```java
public class LazyInitialization {
	
	private static LazyInitialization instance;
	private LazyInitialization () {}
	
	public static LazyInitialization getInstance () {
		if ( instance == null )
			instance = new LazyInitialization();
		return instance;
	}
	
	public void print () {
		System.out.println("It's print() method in LazyInitialization instance.");
		System.out.println("instance hashCode > " + instance.hashCode());
	}
}
```
외부에서 `getInstance()`를 최초로 호출할 때만 인스턴스를 생성하고, 그 이후에는 새로운 인스턴스를 생성하지 않는다. 생성 시점이 뒤로 밀려났다는 장점은 있지만 멀티 스레드 환경에서 동일 시점에 `getInstance()`를 호출한 경우 인스턴스가 2개 이상 생성될 수 있다

### thread safe initialization
```java
public class ThreadSafeInitalization {
	
	private static ThreadSafeInitalization instance;
	private ThreadSafeInitalization () {}
	
	public static synchronized ThreadSafeInitalization getInstance () {
		if (instance == null)
			instance = new ThreadSafeInitalization();
		return instance;
	}
	
	public void print () {
		System.out.println("It's print() method in ThreadSafeInitalization instance.");
		System.out.println("instance hashCode > " + instance.hashCode());
	}
	
}
```
`synchronized`를 사용하여 Thread-safe 하게 구현되었다. 하지만 수 많은 쓰레드가 getInstance()를 호출하면 성능이 저하될 수 있다.

### Initialization on demand holder idiom
```java
public class InitializationOnDemandHolderIdiom {
	
	private InitializationOnDemandHolderIdiom () {}
	private static class Singleton {
		private static final InitializationOnDemandHolderIdiom instance = new InitializationOnDemandHolderIdiom();
	}
	
	public static InitializationOnDemandHolderIdiom getInstance () {
		System.out.println("create instance");
		return Singleton.instance;
	}
}
```

내부 클래스가 필요할 때 동적으로 로딩되기 때문에 `getInstance()`를 호출할 때 내부 클래스가 로드된다. 대부분 이러한 패턴에 따른다

### final 필드와 상수

- final 필드: 한 번 초기화 되면 값이 변경되지 않는다.
- final 메소드: 재정의 하지 못한다
- final 클래스: 상속하지 못한다

상수는 static이면서 final 이어야 한다. static final은 객체마다 저장되지 않고, 클래스에만 포함되며 한 번 초기화되면 값을 변경하지 못한다.

### Annotation
어노테이션(Annotation)은 메타데이터라고 볼 수 있다. 컴파일 과정과 실해 과정에서 코드를 어떻게 컴파일하고 처리할 지 알려주는 정보다. 어노테이션의 용도 세 가지는 아래와 같다
1. 컴파일러에게 코드 문법 에러를 체크하도록 정보 제공
2. 소프트웨어 개발 툴이 빌드나 배치 시 코드를 자동으로 생성할 수 있도록 제공
3. 실행 시 특정 기능을 수행하도록 제공

