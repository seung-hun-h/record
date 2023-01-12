## 1. 추상 클래스
---
- `Animal` 추상 클래스, 이를 상속한 `Cat`, `Penguin` 구체 클래스가 있다

### 자바 - Animal
```Java
public abstract class JavaAnimal {
	protected final String species;
	protected final int legCount;

	public JavaAnimal(String species, int legCount) {
		this.species = species;
		this.legCount = legCount;
	}

	abstract public void move();

	public String getSpecies() {
		return species;
	}

	public int getLegCount() {
		return legCount;
	}
}
```

### 코틀린 - Animal
```Kotlin
abstract class Animal (
	protected val species: String,
	protected val legCount: Int
) {
	abstract fun move();
}
```

### 자바 - Cat
```Java
public final class JavaCat extends JavaAnimal {
	public JavaCat(String species) {
		super(species, 4);
	}

	@Override
	public void move() {
		System.out.println("Cat moves")
	}
}
```

### 코틀린 - Cat
```Kotlin
class Cat(
	species: String
) : Animal(species, 4) {
	override fun move() {
		println("Cat moves")
	}
}
```
- `extends` 를 사용하지 않고 `:`를 사용한다
	- 관용적으로 `:` 의 양쪽은 한 칸씩 띄어준다
- 상위 클래스의 생성자를 바로 호출한다
- 재정의 메서드에 `@Override` 애너테이션을 사용하지 않고 `override` 를 필수적으로 붙여준다

### 자바 - Penguin
```Java
public final class JavaPenguin extends Animal {
	private final int wingCount;

	public JavaPenguin(String species) {
		super(species, 2);
		this.wingCount = 2;
	}

	@Override
	public void move() {
		System.out.println("Penguin moves");
	}	

	@Override
	public int getLegCount() {
		return super.legCount + this.wingCount;
	}
}
```

### 코틀린 - Penguin
- `logCount` 프로퍼티를 오버라이드 해야하므로 추상 클래스 `legCount`에 `open`을 붙여야한다

```Kotlin
abstract class Animal (
	protected val species: String,
	protected open val legCount: Int
) {
	abstract fun move()
}
```


```Kotlin
class Penguin (
	species: String,
) : Animal(species, 2) {
	private val wingCount: Int = 2

	override fun move() {
		println("Penguin moves")
	}

	override val legCount: Int
		get = super.legCount + this.wingCount
}
```

## 2. 인터페이스
- `Flyable`, `Swimmable` 을 구현한 `Penguin`

### 자바 - Flyable, Swimmable
```Java
public interface JavaFlyable {
	default void act() {
		System.out.println("I can fly");
	}
}

public interface JavaSwimmable {
	default void act() {
		System.out.println("I can swim");
	}
}
```

### 코틀린 - Flyable, Swimmable
```Kotlin
interface Flyable {
	fun act() {
		println("I can fly")
	}
}

interface Swimmable {
	fun act() {
		println("I can swim")
	}
}
```
- `default` 키워드 없이 메서드 구현이 가능하다

### 자바 - Penguin
```Java
public final class JavaPenguin extends Animal implements Flyable, Swimmable {
	@Override
	public void act() {
		JavaSwimmable.super.act();
		JavaFlyable.super.act();
	}
}
```

### 코틀린 - Penguin
```Kotlin
class Penguin(
	species: String
) : Animal(species, 2), Swimmable, Flyable {
	override fun act() {
		super<Swimmable>.act()
		super<Flyable>.act()
	}
}
```
- `implements`를 사용하지 않고 상속때와 동일하게 `:`를 사용한다
- 중복되는 인터페이스를 특정할 때 `super<타입>.함수`를 사용한다

### backing field
- 코틀린에서는 backing field가 없는 프로퍼티를 인터페이스에 만들 수 있다
```Kotlin
interface Swimmable {
	val swimmablity: Int // 구현 클래스에서 재정의하길 기대

	fun act() {
		println("I can swim")
	}
}

class Penguin(
	species: String
) : Animal(species, 2), Swimmable, Flyable {
	override fun act() {
		super<Swimmable>.act()
		super<Flyable>.act()
	}

	override val swimmablity: Int
		get() = 2
}
```

## 3. 클래스를 상속할 때 주의점
```Kotlin
open class Base (
	open val number: Int = 100
) {
	init {
		println("Base Class")
		println(number)
	}
}

class Derived (
	override val number: Int
) : Base(number) {
	init {
		println("Derived Class")
	}
}
```

```Kotlin
fun main() {
	Derived(300)
}

/**
Base Class
0
Derived Class
*/
```
- `number`가 300도 아니고, 100도 아니고, 0이 출력됐다
- 상위 클래스의 초기화가 먼저 이루어져, 하위 클래스의 `number`의 값이 초기화 되지 않았기 때문에 0이 출력됐다
- 따라서 상위 클래스의 constructor, init 은 하위 클래스의 재정의 필드에 접근하면 안된다
- **상위 클래스를 설계할 때 생성자 또는 초기화 블록에 사용되는 프로퍼티에는 open을 피해야 한다**


## 4. 상속 관련 지시어 정리
- final: override 할 수 없게 한다. default로 보이지 않게 존재한다
- open: override를 열어준다
- abstract: 반드시 override 해야 한다
- override: 상위 타입을 재정의하고 있다


