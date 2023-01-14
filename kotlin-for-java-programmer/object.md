## 1. static 함수와 변수
---
### 자바 - Person
```Java
public class JavaPerson {
	private static final int MIN_AGE = 1;

	public static JavaPerson newBaby(String name) {
		return new JavaPerson(name, MIN_AGE);
	}

	private String name;
	private int age;

	private JavaPerson(String name, int age) {
		this.name = name;
		this.age = age;
	}
}
```

### 코틀린 - Person
```Kotlin
class Person private constructor (
	private val name: String,
	private val age: Int,
) {

	companion object { // 1
		private val MIN_AGE = 0 // 2
		fun newBaby(name: String): Person {
			return Person(name, MIN_AGE)
		}
	}
}
```
1. `static` 대신 `companion object`를 사용했다
   - `companion object`: 클래스와 동행하는 유일한 오브젝트
2. `val`을 사용하면 런타임에 변수가 할당된다
   - `const val`을 사용하면 컴파일 타임에 변수가 할당된다
   - 진짜 상수를 선언할 때는 `const val`을 사용하자

### companion object
- 동반객체도 하나의 객체로 간주된다
	- 이름을 붙일 수도 있고, 인터페이스를 구현할 수도 있다
```Kotlin
class Person private constructor (
	private val name: String,
	private val age: Int,
) {

	companion object Factory : Log { // 1
		private val MIN_AGE = 0
		fun newBaby(name: String): Person {
			return Person(name, MIN_AGE)
		}

		override fun log() {
			println("Log")
		}
	}
}
```
1. `Log` 인터페이스를 구현한다

- 자바에서 코틀린 동반 객체를 사용하기 위해서는 `@JvmStatic`을 붙여야 한다

```Kotlin
class Person private constructor (
	private val name: String,
	private val age: Int,
) {

	companion object Factory : Log { // 1
		private val MIN_AGE = 0
		
		@JvmStatic
		fun newBaby(name: String): Person {
			return Person(name, MIN_AGE)
		}

		override fun log() {
			println("Log")
		}
	}
}
```

```Java
Person person = Person.newBaby("A");
```

- 동반객체에 이름이 있으면 이름을 사용하면 된다
```Java
Person person = Person.Factory.newBaby("A");
```

## 2. 싱글톤
---
### 자바 싱글톤
```Java
public class SingleTon {
	private static final INSTANCE = new SingleTon();

	private JavaSingleTon() {}

	public static SingleTon getInstance() {
		return INSTANCE;
	}
}
```

### 코틀린 싱글톤
```Kotlin
object SingleTon
```

## 3. 익명 클래스
---
### 자바 익명 클래스
```Java
public class Main {
	public static void main(String[] args) {
		moveSomething(new Movable() {
			@Override
			public void move() { 
				System.out.println("move"); 
			}

			@Override
			public void fly() {
				System.out.println("fly");
			}	
		});
	}

	private static void moveSomething(Movable, movable) {
		movable.move();
		movable.fly();
	}
}
```

### 코틀린 익명 클래스

```Kotlin
fun main() {
	moveSomething(object : Movable {
		override fun move() {
			println("move")
		}

		override fun fly() {
			println("fly")
		}
	})
}
```
- `object : 타입`으로 익명 클래스를 구현할 수 있다
