## 1. 함수 선언 문법
---
### 자바 - 더 큰 수 구하기
```Java
public int max(int a, int b) {
	if (a > b) {
		return a;
	}
	return b;
}
```

### 코틀린 - 더 큰 수 구하기1
```Kotlin
public fun max(a: Int, b: Int): Int {
	if (a > b) {
		return a
	}
	return b
}
```

### 코틀린 - 더 큰 수 구하기2
- 코틀린의 if문은 Expression이다
- 함수의 기본 접근제어자가 public 이므로 생략 가능하다

```Kotlin
fun max(a: Int, b: Int): Int {
	return if (a > b) {
		a
	} else {
		b
	}
}
```

### 코틀린 - 더 큰 수 구하기3
- 함수가 하나의 결과값이면 block 대신 `=` 사용 가능
- 개행 없이 한 줄로 변경 가능
- `=` 을 사용하는 경우 반환  타입 생략 가능
	- block을 사용하는 경우에는 반환 타입이 Unit이 아닌경우에는 명시해야 한다

```Kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

### 함수 선언 위치
- 함수는 클래스 안, 파일 최상단에 위치할 수 있다
- 한 파일에 여러 함수들을 작성할 수도 있다

## 2. default parameter
---
- 함수의 파라미터에 기본값을 줄 수 있다
	- 자바는 기본값을 줄 수 없어 오버로딩 해야 한다
	- 코틀린에서도 오버로딩을 할 수는 있다
```Kotlin
fun repeat(
	str: String,
	num: Int = 3,
	useNewLine: Boolean = true
) {
	for (i in 1..num) {
		if (useNewLine) {
			println(str)
		} else {
			print(str)
		}
	} 
}
```

## 3. named argument(parameter)
---
- 함수를 호출할 때 전달할 인자의 이름을 명시할 수도 있다
	- default parameter 예제에서 `useNewLine`은 false를, `num`은 기본값을 사용하고 싶은 경우에 유용하다

```Kotlin
fun main() {
	repeat("Hello world", useNewLine = false)
}
```
- builder를 사용하지 않고 builder의 장점을 얻을 수 있다
	- 같은 타입의 인자가 여러개일 경우 이름을 사용하여 인자를 구분할 수 있다
- 코틀린에서 자바 함수를 가져다 사용할 때는 named parameter를 사용할 수 없다
	- 자바에서는 named parameter를 제공하지 않기 때문이다

## 4. 같은 타입의 여러 파라미터 받기(가변인자)
---
### 자바의 가변인자
```Java
public static void printAll(String... strings) {
	for (String str : strings) {
		System.out.println(str);
	}
}

public static void main(String[] args) {
	String[] array = new String[] {"A", "B", "C"};
	printAll(array);

	printAll("A", "B", "C");
}
```

### 코틀린의 가변인자
```Kotlin
fun printAll(vararg strings: String) {
	for (str in strings) {
		println(str)
	}
}

fun main() {
	val array = arrayOf("A", "B", "C")
	printAll(*array)

	printAll("A", "B", "C")
}
```
	- `*` 은 spread 연산자
