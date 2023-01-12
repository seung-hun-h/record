## 1. if문
---
- 사용 방법은 자바와 동일하다
```Kotlin
if (조건1) {
...
} else if (조건2) {
...
} else {
...
}
```
- 자바와 차이점
		- 자바에서 `if`문은 Statement이지만, 코틀린에서는 Expression이다

## 2. Expression과 Statement
- Statement
	- 프로그램의 문장, 하나의 값으로 도출되지 않는다
	- 실행가능한 최소의 독립적인 코드 조각을 말한다
	- Statement는 최소 한 개 이상의 Expression과 프로그래밍 키워드를 포함하는 경우가 많다
	- `val number = 1`이라는 Statement은 어떤 값을 반환하지 않기 때문에 Expression이 아니다
- Expression
	- 하나의 값으로 도출되는 문장
	- `1 + 2` 같은 사칙연산식이 대표적인 Expression이다
- Expression는 Statement에 포함된다

- 자바의 `if-else`는 Expression이 아니다
```Java
public String passOrFail(int score) {
	return if (score > 50) { // 컴파일 에러가 발생한다
		return "P";
	} else {
		return "F";
	}
}
```
- 자바의 3항 연산자는 Expression이다
```Java
public String passOrFail(int score) {
	return score > 50 ? "P" : "F";
}
```

- 코틀린의 `if-else`는 Expression이다
	- 따라서 코틀린은 3항 연산자가 없다(필요 없기때문).
```Kotlin
fun passOrFail(score: Int): String {
	return if (score > 50) {
		"P"
	} else {
		"F"
	}
}
```


## 3. switch와 when
- 코틀린의 when은 자바의 switch와 유사하다
```Kotlin
fun getGradeWithWhen(score: Int): String {
	return when(score / 10) {
		9 -> "A"
		8 -> "B"
		7 -> "C"
		else -> "D"
	}
}

fun getGradeWithWhen(score: Int): String {
	return when(score) {
		in 90..99 -> "A"
		in 80..89 -> "B"
		in 70..79 -> "C"
		else -> "D"
	}
}
```
