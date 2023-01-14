## 1. 확장 함수
---
- 확장 함수는 어떤 클래스 안에 있는 메서드처럼 호출 될 수 있지만, 클래스 외부에서 정의된 함수를 말한다

```Kotlin
fun String.lastChar(): Char { // 1
	return this[this.length - 1] // 2
}
```
1. `String`의 확장 함수
   - 이때 String은 수신 객체 타입이라 한다
2. `this`를 통해 인스턴스에 접근할 수 있다
   - 이때 this는 수신 객체라 한다

### 캡슐화
- 확장함수는 캡슐화를 보장하기 위해 `private` 혹은 `protected` 멤버를 가져올 수 없다

### 멤버 함수와 시그니처가 같을 경우
- 멤버 함수가 우선적으로 호출된다
- 확장함수를 먼저 만들고, 이후에 같은 시그니처의 멤버 함수가 생긴다면 오류가 발생할 수 있다

### 확장 함수의 재정의
```Kotlin
open class Train (
	val name: String = "새마을 기차",
	val price: Int = 5_000
)

fun Train.isExpensive(): Boolean {
	println("Train의 확장함수")
	return this.price >= 10_000
}

class Srt: Train("SRT", 40_000)

fun Srt.isExpensive(): Boolean {
	println("Srt의 확장함수")
	return this.price >= 10_000
}
```

```Kotlin
val train: Train = Train()
train.isExpensive() // Train의 확장함수

val train: Train = Srt()
train.isExpensive() // Train의 확장함수

val train: Srt = Srt()
train.isExpensive() // Srt의 확장함수
```
- 해당 변수의 현재 타입에 따라 호출될 확장함수가 결정된다

### 자바에서 코틀린 확장 함수 호출
- 정적 메서드를 호출하는 것처럼 사용할 수 있다
```Java
public static void main(String[] args) {
	StringUtilsKt.lastChar("ABC");
}
```

### 확장 프로퍼티
- 확장 함수의 개념은 확장 프로퍼티로 확장된다

```Kotlin
fun String.lastChar(): Char {
	return this[this.length - 1]
}

val String.lastChar: Char
	get() = this[this.length - 1]
 ```

## 2. infix 함수
---
- 중위 함수는 함수를 호출하는 새로운 방법이다
	- `변수 함수이름 argument`

```Kotlin
infix fun Int.add(other: Int) {
	return this + other
}

fun main() {
	3 add 4
}
```


## 3. inline 함수
---
- 함수가 호출되는 대신, 함수를 호출한 지점에 함수 본문을 그대로 복붙하고 싶은 경우 사용한다

<img width="700" alt="image" src="https://user-images.githubusercontent.com/60502370/212466705-0b067da0-150b-405a-93c6-5c363387c703.png">
- 함수를 파라미터로 전달할 때 오버헤드를 줄일 수 있다

## 4. 지역 함수
---
- 함수안의 함수를 말한다

```Kotlin
fun createPerson(firstName: String, lastName: String): Person {
	fun validateName(name: String, fieldName: String) {
		if (name.isEmpty()) {
			throw IllegalArgumentException("${fieldName}은 비어있을 수 없습니다. 현재 값: $name")
		}
	}

	validateName(firstName, "firstName")
	validateName(lastName, "lastName")

	return Person(firstName, lastName, 1)
}
```

- 함수로 추출하면 좋을 것 같은데, 이 함수를 지금 함수 내에서만 사용하고 싶을때 사용한다
	- depth가 깊어지기도 하고 코드가 깔끔하지 않다
