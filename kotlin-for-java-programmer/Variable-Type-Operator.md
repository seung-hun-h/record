## 1. 변수
### var과 val
- var은 가변이고 val은 불변이다
```Java
long number1 = 10L;
final long number2 = 10L;
```

```Kotlin
var number1 = 10L
val number2 = 10L
```
- var은 처음 변수를 할당하고 재할당할 수 있지만, val은 처음 할당 후 재할당 할 수 없다

### 타입추론
- 변수에 값을 할당하면 컴파일러가 타입을 추론할 수 있다
- `변수명:타입` 으로 타입을 명시할 수도 있다

```Kotlin
var number1: Long = 10L
```

### Primitive Type
- 자바에서는 `long`, `Long` 처럼 원시 타입과 참조  타입의 표현이 다르다
- 코틀린은 원시 타입과 참조 타입의 표현이 동일하다
> Some types can have a special internal representation - for example, numbers, characters and booleans - can be represented as primitive values at runtime - but to the user they look like ordinary classes.

- 참조 타입을 연산에 사용하면 성능 저하가 일어날 수 있으므로, 컴파일러가 내부적으로 원시 타입으로 변경한다
- 개발자는 원시 타입과 참조  타입을 신경쓰지 않고 코드를 작성하면 된다

### nullable 변수
- 자바의 참조 타입은 null이 될 수 있다
- 코틀린은 기본적으로 null을 허용하지 않는다
- nullable 변수가 따로 있는데 `타입?`를 사용한다

```Kotlin
var number1 = null(X)
var number2:Long? = null
```

### 인스턴스화
- 자바는 인스턴스화 하기위해 `new`를 사용한다
- 코틀린은 `new`를 생략한다
```Java
Person person = new Person("name", 10);
```

```Kotlin
val person = Person("name", 10)
```
---
## 2. 코틀린에서 null을 다루는 방법
### 코틀린에서 null 체크
- 자바 예제
```Java
public boolean startsWithA1(String str) {
	if (str == null) {
		throw new IllegalArgumentException("str should not be null");
	}

	return str.startsWith("A");
}
```

```Java
public Boolean startsWithA2(String str) {
	if (str == null) {
		return null;
	}

	return str.startsWith("A");
}
```

```Java
public boolean startsWithA3(String str) {
	if (str == null) {
		return false;
	}

	return str.startsWith("A");
}
```

- 코틀린 코드
```Kotlin
fun startsWithA1(str: String?): Boolean {
	if (str == null) {
		throw IllegalArgumentException("str should not be null")
	}

	return str.startsWith("A")
}
```

```Kotlin
fun startsWithA2(str: String?): Boolean? {
	if (str == null) {
		return null
	}

	/*
		위에서 null check을 해주면 컴파일러가 그 다음 코드에는 null 이 아님을 추론한다
	*/
	return str.startsWith("A")
	
}
```

```Kotlin
fun startsWithA3(str: String?): Boolean {
	if (str == null) {
		return false
	}

	return str.startsWith("A")
}
```

### Safe Call과 Elvis 연산자
- 코틀린에서는 null이 가능한 타입을 완전히 다르게 취급한다
- Safe Call은 null이 아니면 실행하고, null이면 그대로 null을 반환한다

```Kotlin
val str: String? = "ABC"
str.length // 불가능
str?.length // Safe Call
```

- Elvis 연산자는 앞의 연산 결과가 null이면 뒤의 값을 사용한다
```Kotlin
val str: String? = "ABC"
str?.length ?: 0 // str이 null이면 0을 사용
```

### Safe Call과 Elvis 연산자를 사용해 코틀린스럽게 코드 작성하기

```Kotlin
fun startsWithA1(str: String?): Boolean {
	return str?.startsWith("A")
		?: throw IllegalArgumentException("str should not be null")
}
```

```Kotlin
fun startsWithA2(str: String?): Boolean? {
	return str?.startsWith("A")
}
```

```Kotlin
fun startsWithA3(str: String?): Boolean {
	return str?.startsWith("A") ?: false
}
```

- 응용으로 Early Return에도 사용할 수 있다

```Kotlin
fun calculate(number: Long?): Long {
	number ?: return 0
	// 다음 로직
}
```

### 널 아님 단언 !!
- nullable 타입이지만 null이 아님을 확신할 수 있는 경우 `!!`를 사용할 수 있다

```Kotlin
fun startsWithA1(str: String?): Boolean {
	return str!!.startsWith("A")
}
```
- 하지만 null인 경우 NPE가 발생하기 때문에 주의 해야 한다

### 플랫폼 타입
- 다른 프로그래밍 언어에서 넘어온 타입들을 플랫폼타입이라 한다
- 자바 코드에서 어떤 값이 Nullable 하다면 `@Nullable` 애너테이션을 자바 코드에 추가하고, Nullable하지 않다면 `@NotNull`과 같은 애너테이션을 추가하면, 코틀린 컴파일러가 타입을 추론 할 수 있다

- `@Nullabe`
```Java
public class Person {
	private final String name;

	public Person(String name) {
		this.name = name;
	}

	@Nullabe
	public String getName() {
		return name;
	}
}
```

```Kotlin
fun main() {
	val person = Person("이름")
	startsWithA(person.name) // x
	startsWithA(person?.name) // o
}

fun startsWithA(str: String): Boolean {
	return str.startsWith("A")
}
```

- `@NotNull`
```Java
public class Person {
	private final String name;

	public Person(String name) {
		this.name = name;
	}

	@NotNull
	public String getName() {
		return name;
	}
}
```

```Kotlin
fun main() {
	val person = Person("이름")
	startsWithA(person.name)
}

fun startsWithA(str: String): Boolean {
	return str.startsWith("A")
}
```

- 없음
```Java
public class Person {
	private final String name;

	public Person(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}
}
```

```Kotlin
fun main() {
	val person = Person("이름")
	startsWithA(person.name)
}

fun startsWithA(str: String): Boolean {
	return str.startsWith("A")
}
```
- 애너테이션 정보가 없다면 컴파일 에러가 아닌 런타임 에러가 발생할 수 있다
	- 컴파일러가 잡지 못한다
---
## 3. 코틀린에서 Type을 다루는 방법
### 기본 타입
- 코틀린의 기본 타입
	- Byte
	- Short
	- Int
	- Long
	- Float
	- Double
- 코틀린은 선언된 기본값을 보고 타입을 추론한다
- 코틀린에서 타입 변환은 명시적으로 이루어져야 한다
	- 자바는 암시적으로 이루어진다
```Java
int number1 = 4;
long number2= number1;
```

```Kotlin
val number1 = 4

// Type mismatch
val number2: Long = number1 

val number3: Long = number1.toLong()
```
- `to변환타입()` 를 사용해 명시적으로 타입을 변환한다

### 타입 캐스팅
- 자바 예제
```Java
public static void printAgeIfPerson(Object obj) {
	if (obj instanceof Person) {
		Person person = (Person) obj;
		System.out.println(person.getAge());
	}
}
```
- 코틀린 에제
```Kotlin
fun printAgeIfPerson(obj: Any) {
	if (obj is Person) {
		val person = obj as Person
		println(person.age)
	}
}
```
- `instanceof` -> `is`
- `(Person)` -> `as Person`
	- 위에서 타입 검사를 해주었기 때문에 `as Person`은 생략가능하다
```Kotlin
if (obj is Person) {
	println(obj.age) // 스마트 캐스트
}
```


#### is
- `value is Type`
	- value가 Type이면 true
	- value가 Type이 아니면 false
- `is`의 반대는 `!is`

#### as
- `value as Type`
	- value가 Type이면 캐스팅
	- value가 Type이 아니면 예외 발생
#### as?
- `value as? Type`
	- value가 Type이면 캐스팅
	- value각 null이면 null
	- value가 Type이 아니면 null

### Kotlin의 특이한 타입
#### Any
- Java의 Object역할
- 모든 Primitive Type의 최상위 타입
- Any 자체는 null을 포함할 수 없어, null을 포함하고 싶다면 Any?
- equals, hashCode, toString 존재

#### Unit
- Java의 void와 동일한 역할
- void와 다르게 그 자체로 타입 인자로 사용가능하다
- 함수형 프로그래밍에서 Unit은 단 하나의 인스턴스만 갖는 타입을 의미. 코틀린의 Unit은 실제 존재하는 타입이라는 것을 표현

#### Nothing
- 함수가 정상적으로 끝나지 않았다는 사실을 표현하는 역할
- 무조견 예외를 반환하는 함수, 무한 루프 함수 등

### String interpolation / String indexing
- `${변수}`를 사용하면 값이 들어간다
```Kotlin
println("사람의 이름은 ${person.name}이고 나이는 ${person.age}이다.")
```
- `$변수`를 사용할 수 있지만, 유지보수 측면에서 `${변수}`를 사용하는 것이 좋다
	- 가독성
	- 일괄 변환
	- 정규식 활용
- `trimIndent()`와 `""" """`를 사용하면 문자열 표현이 간편하다
```Kotlin
val withoutIndent = 
"""
	ABC
	123
	456
""".trimIndent()

println(withoutIndent)
/* 인덴트가 제거된 상태로 출력
ABC
123
456
*/
```
- `[index]`로 특정 문자를 가져올 수 있다

```Kotlin
val str = "ABCDE"
val ch = str[1]
```
----
## 4. 코틀린에서 연산자를 다루는 방법
### 단항 연산자와 산술 연산자
- 단항 연산자
	- `++`
	- `--`
- 산술 연산자
	- `+`
	- `-`
	- `/`
	- `%`
- 산술대입 연산자
	- `+=`
	- `-=`
	- `*=`
	- `/=`
	- `%=`
- 모두 자바와 동일하다

### 비교 연산자와 동등성, 동일성
- 비교 연산자
	- `>`, `<`, `>=`, `<=`
	- 사용방법은 자바와 동일하다
	- 하지만 자바와 다르게 객체를 비교할 때 자동으로 `compareTo`를 호출해준다
- 동등성과 동일성
	- 자바는 동일성 비교에 `==`, 동등성 비교에 `equals`를 사용한다
	- 코틀린은 동일성 비교에 `===`, 동등성 비교에 `==`를 사용한다

### 논리연산자
- 논리 연산자
	- `&&`
	- `||`
	- `!`
- 자바와 동일하다

### 코틀린에만 있는 특이한 연산자
- `in`, `!in`
	- 컬렉션이나 범위에 포함되어있다, 포함되어 있지 않다
- `a..b`
		- a부터 b까지의 범위 객체를 생성한다
- `a[i]`
	- a에서 i번째 원소를 가져온다

### 연산자 오버로딩
- 코틀린에서는 객체마다 연산자를 직접 정의할 수 있다
```Kotlin
val money1 = Mone(1_000L)
val money1 = Mone(2_000L)

println(money1 + money2) // Money(amount=3000)
```
