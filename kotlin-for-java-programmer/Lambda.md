## 1. 코틀린에서의 람다
---
- 코틀린에서는 함수가 그 자체로 값이 될 수 있다
	- 변수에 할당할 수도 있고, 파라미터로 넘길수도 있다

### 람다를 만드는 방법
```Kotlin
// 람다 만드는법1
val isApple = fun(fruit: Fruit): Boolean {
	return fruit.name == "사과"
}

// 람다 만드는법2
val isApple2 = { fruit: Fruit -> fruit.name == "사과" }

// 람다 호출하는법1
isApple(Fruit("사과", 1000))

// 람다 호출하는법2
isApple.invoke(Fruit("사과", 1000))
```

### 람다의 타입
```Kotlin
// 람다 만드는법1
val isApple: (Fruit) -> Boolean = fun(fruit: Fruit): Boolean {
	return fruit.name == "사과"
}

// 람다 만드는법2
val isApple2: (Fruit) -> Boolean = { fruit: Fruit -> fruit.name == "사과" }
```
- `(파라미터 타입, ...) -> 반환 타입`으로 람다의 타입을 명시할 수 있다

### 여러가지 람다 표현법
```Kotlin
private fun filterFruits(fruits: List<Fruit>, filter: (Fruit) -> Boolean): List<Fruit> {

	val results = mutableList<Fruit>()
	for (fruit in fruits) {
		if (filter(fruit)) {
			results.add(fruit)
		}
	}

	return results
}
```

```Kotlin
filterFruits(fruits, { fruit: Fruit -> fruit.name == "사과" })

filterFruits(fruits) { fruit: Fruit -> fruit.name == "사과" } // 1
filterFruits(fruits) { fruit -> fruit.name == "사과" } // 2
filterFruits(fruits) { it.name == "사과" } // 3

filterFruits(fruits) { fruit -> // 4
	pritln("사과만 받는다")
	fruit.name == "사과" 
}
```
1. 마지막 파라미터가 함수인 경우 소괄호 밖에 람다를 사용할 수 있다
2. 파라미터 타입을 생략할 수 있다
3. 람다의 파라미터가 하나인경우 `it`로 직접 참조할 수 있다
4. 람다를 여러 줄 작성할 수 있고, 마지막 줄의 결과가 람다의 반환값이다

## 2.  Closure
---
- 자바에서는 람다를 사용할 때 변수는 final이거나 사실상 final이어야 한다
```Java
String targetFruitName = "바나나";
targetFruitName = "수박";
filterFruits(fruits, (fruit) -> targetFruitName.equals(fruit.getName())); // Exception!!
```

- 코틀린은 아무 문제없이 동작한다
```Kotlin
var targetFruitName = "바나나"
targetFruitName = "수박"
filterFruits(fruits) { it.name == targetFruitName }
```
- 코틀린에서는 람다가 시작하는 지점에 참조하고 있는 변수들을 모두 포획하여 그 정보를 가지고 있다
- 이렇게 해야만 람다를 진정한 일급 시민으로 간주할 수 있고, 이러한 데이터 구조를 Closure라 한다

## 3. try-with-resources 다시 보기
---

```Kotlin
fun readFile(path: String) {
	BufferedReader(FileReader(path)).use { reader ->
		println(reader.readLine())
	}
}
```

### use
```Kotlin
public inline fun <T : Closeabe?, R> T.use(block: (T) -> R): R {
...
}
```
- Closeable을 구현한 타입 T를 파라미터로 받고, R 타입을 반환하는 람다를 파라미터로 받는다