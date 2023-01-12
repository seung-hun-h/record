## 1. 클래스와 프로퍼티
---
### 자바 예제
```Java
public class JavaPerson {
	private final String name;
	private int age;

	public JavaPerson(String name, int age) {
		this.name = name;
		this.age = age;
	}

	public String getName() {
		return name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}
}
```

### 코틀린 예제1
```Kotlin
public class Person constructor(name: String, age: Int) {
	val name = name
	var age = age
}
```

### 코틀린 예제2
- `constructor`, `public` 은 생략할 수 있다
- 클래스 필드 선언과 생성자를 동시에 선언할 수 있다
```Kotlin
class Person(
	val name: String,
	var age: Int
)
```

### 코틀린 프로퍼티
- 자바에서는 getter, setter 를 코드로 작성에 필드에 접근한다
- 코틀린에서는 `.필드`를 통해 getter, setter 를 호출한다

```Kotlin
fun main() {
	val person = Person("Ham", 10)
	println("${person.name}의 나이는 ${person.age}입니다.")
	person.age = 20
}
```
- 자바 코드를 가져와 쓸 때도 `.필드`를 통해 getter, setter를 호출한다
```Java
fun main() {
	val javaPerson = JavaPerson("Ham", 10)
	println("${person.name}의 나이는 ${person.age}입니다.")
	person.age = 20
}
```

## 2. 생성자와 init
---
### 자바의 객체 생성시점 검증
- 생성자에 검증 로직을 작성하면된다
```Java
public class JavaPerson {
	private final String name;
	private int age;

	public JavaPerson(String name, int age) {
		if (age <= 0) {
			throw new IllegalArgumentException(String.format("Age should not be %d", age));
		}
		this.name = name;
		this.age = age;
	}
	...
}
```

### 코틀린의 객체 생성시점 검증
- `init`을 사용하면 객체 생성시점에 검증할 수 있다
- `init` 블록은 생성자가 호출되는 시점에 호출된다

```Kotlin
class Person(
	val name: String,
	var age: Int
) {
	init {
		if (age <= 0) {
			throw IllegalArgumentException("Age should not be ${age}")
		}
	}
}
```

### 자바의 생성자 오버로딩
```Java
public class JavaPerson {
	private final String name;
	private int age;

	public JavaPerson() {
		this("no name");
	}

	public JavaPerson(String name) {
		this(name, 1);
	}

	public JavaPerson(String name, int age) {
		if (age <= 0) {
			throw new IllegalArgumentException(String.format("Age should not be %d", age));
		}
		this.name = name;
		this.age = age;
	}
	...
}
```

### 코틀린의 생성자 오버로딩
```Kotlin
class Person ( // 1
	val name: String,
	var age: Int
) {
	init {
		println("init 블록") // 2
	}

	constructor(name: String) : this(name, 1) { // 3
		println("부 생성자1")
	}
	
	constructor() : this("no name") { // 4
		println("부 생성자2")
	}
	
}
```
1. 주 생성자(Primary Constructor)이다
   - 반드시 존재해야 한다
   - 주 생성자에 파라미터가  하나도 없다면 생략 가능하다
2. 초기화 블록이다
3. 부 생성자(Secondary Constructor)이다. 주 생성자를 호출한다
   - 부 생성자는 `this`를 사용해 주 생성자 혹은 부 생성자를 호출해야 한다
4. 부 생성자이다. 부 생성자1을 호출한다

```Kotlin
fun main() {
	Person()
}
```
- 부 생성자2 를 호출하면
	- 2, 3, 4 순서로 출력이 된다

### 부 생성자 보다는 Default Parameter
- 부 생성자 보다는 Default Parameter를 권장한다
```Kotlin
class Person (
	val name: String = "no name",
	var age: Int
) {
	...
}
```

- 하지만 Converting 같은 경우에는 부 생성자를 사용할 수 있지만, '정적 팩토리 메서드'를 사용하는 것이 추천된다

## 3. 커스텀 getter, setter
---
### 자바의 커스텀 getter
```Java
public boolean isAdult() {
	return this.age >= 20;
}
```

### 코틀린의 커스텀 getter - 함수
```Kotlin
fun isAdult(): Boolean {
	return this.age >= 20
}
```

### 코틀린의 커스텀 getter - 프로퍼티
```Kotlin
// 1
val isAdult: Boolean
	get() = this.age >= 20

// 2
val isAdult: Boolean
	get() {
		return this.age >= 20
	}
```

- 객체의 속성이라면 커스텀 getter 를 사용하고, 그렇지 않으면 함수를 사용하는 것이 좋겠다

### 코틀린의 커스텀 getter - 값 변형
- `name`을 대문자로 변형
```Kotlin
class Person (
	name: String,
	var age: Int
) {
	val name: String = name // 1
		get() = field.upperase()
}
```
1. 주 생성자에서 받은 name을 불변 프로퍼티 name에 바로 대입
2. 항상 대문자를 반환하도록 커스텀 getter 정의
 
### 코틀린의 커스텀 setter
```Kotlin
class Person (
	name: String,
	var age: 
) {
	var name: String = name
		set(value) {
			field = value.uppercase()
		}
}
```
- setter는 잘 사용하지 않는다


## 4. backing field
```Kotlin
class Person (
	name: String,
	var age: Int
) {
	val name: String = name
		get() = field.upperase() // 1
}
```
1. name에 대한 커스텀 getter를  만들때는 `field`를 사용
   - 만약 `name.uppercase()` 로 작성하면 다시 getter를 호출하는 것이므로 무한 루프가 된다
   - `field`는 무한 루프를 막기위한 예약어이다

### backing field 대신 this
- backing field 대신 this를 사용하면 해결할 수 있다

```Kotlin
class Person (
	name: String,
	var age: Int
) {
	val name: String = name
		get() = this.upperase()
}
```