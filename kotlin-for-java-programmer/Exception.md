## 1. try-catch-finally
---
### 자바의 try-catch
```Java
private int parseIntOrThrow(@NotNull String str) {
	try {
		return Integer.parseInt(str);
	} catch (NumberFormatException e) {
		throw new IllegalArgumentException(String.format("%s is not a number", str));
	}
}
```

```Java
private Integer parseIntOrThrow(@NotNull String str) {
	try {
		return Integer.parseInt(str);
	} catch (NumberFormatException e) {
		return null;
	}
}
```

### 코틀린의 try-catch
- 자바와 사용 방법이 거의 같다
	- 단, 코틀린의 try-catch는 Expression이다
```Kotlin
fun parseIntOrThrow(str: String): Int {
	try {
		return str.toInt()
	} catch (e: NumberFormatException) {
		throw IllegalArgumentException("${str} is not a number", str);
	}
}

fun parseIntOrThrow(str: String): Int {
	return try {
		str.toInt()
	} catch (e: NumberFormatException) {
		null
	}
}
```

## 2. Checked Exception과 Unchecked Exception
---
### 자바 예제 - 파일 읽기
```Java
public void readFile(String path) throw IOException {
	File file = new File(path);
	BufferedReader reader = new BufferedReader(new FileReader(file));
	System.out.println(reader.readLine());
	reader.close();
}
```
- `IOException`은 Checked Exception 이므로 호출하는 쪽에서 잡아서 처리하거나, 밖으로 던져야한다

### 코틀린 - 파일 읽기
```Kotlin
fun readFile(path: String) {
	val file = File(path)
	val reader = BufferedReader(FileReader(file))
	println(reader.readLine())
	reader.close()
}
```
- 코틀린에는 Checked Exception이 없다. 모두 Unchecked Exception 이다

## 3. try-with-resources
---
### 자바 예제 - 파일 읽기
```Java
public void readFile(String path) throw IOException {
	try (BufferedReader reader = new BufferedReader(new FileReader(file))){
		System.out.println(reader.readLine());
	}
}
```
- jdk7 부터 추가된 try-with-resources 를 사용하면 수동으로 리소스를 닫아주지 않아도된다

### 코틀린 - 파일 읽기
```Kotlin
fun readFile(path: String) {
	BufferedReader(FileReader(path)).use { reader ->
		println(reader.readLine())
	}
}
```
- 코틀린에는 `try-with-resources` 구문이 없다. 대신 `use` 라는 inline 확장 함수를 사용한다