## 1. 자바와 코틀린의 가시성 제어
---
### 자바의 가시성 제어
| Access Modifier | Description              |
| --------------- | ------------------------ |
| public          | 모든 곳                  |
| protected       | 같은 패키지, 하위 클래스 |
| default         | 같은 패키지              |
| private         | 선언된 클래스 내부       |
- 자바의 기본 접근 지시어는 default이다

### 코틀린의 가시성 제어
| Access Modifier | Description                |
| --------------- | -------------------------- |
| public          | 모든 곳                    |
| protected       | 선언된 클래스, 하위 클래스 |
| internal        | 같은 모듈                  |
| private         | 선언된 클래스 내부         |

- 코틀린에서는 패키지를 namespace를 관리하기 위한 용도로 사용되고, 가시성 제어에는 사용되지 않는다
- 모듈은 한 번에 컴파일되는 코틀린 코드의 집합을 말한다
	- IDEA Module
	- Maven Project
	- Gradle Source Set
- 코틀린의 기본 접근 지시어는 public 이다

## 2. 코틀린 파일의 접근 제어
---
- 코틀린은 `.kt` 파일에 변수, 함수, 클래스 여러개를 바로 만들 수 있다

| Access Modifier | Description                |
| --------------- | -------------------------- |
| public          | 모든 곳                    |
| protected       | 파일에는 사용 불가 |
| internal        | 같은 모듈                  |
| private         | 선언된 클래스 내부         |

- 코틀린은 패키지를 namespace를 구분하기 위해 사용하기 때문에 파일에는 사용할 수 없다

## 3. 다양한 구성요소의 접근 제어
---

### 클래스 멤버
| Access Modifier | Description                |
| --------------- | -------------------------- |
| public          | 모든 곳                    |
| protected       | 선언된 클래스, 하위 클래스 |
| internal        | 같은 모듈                  |
| private         | 선언된 클래스 내부         |

### 생성자
- 클래스 멤버와 동일하지만 `access-modifier constructor`을 따라야 한다
```Kotlin
class Cat internal constructor (
	val price: Int
)
```
- 유틸 함수는 파일에 함수로 구현하면 편리하다

```Kotlin
// StringUtilsKt.kt
fun isDirectoryPath(path: String): Boolean {
	return path.endsWith("/")
}
```
- 위 코드를 컴파일하면 자바 바이트 코드로는 아래와 같이 컴파일된다
```Java
public final class StringUtilsKt {
	private StringUtilsKt() {}

	public static boolean isDirectoryPath(String path) {
		return path.endsWith("/");
	}
}
```

### 프로퍼티
- 가시성의 범위는 클래스 멤버와 동일하다
```Kotlin
class Car (
	internal val name: String, // 1
	private var owner: String, // 2
	_price: Int
) {
	var price = _price
		private set // 3
}
```
1. name의 getter, setter는 모두 internal
2. owner의 getter, setter는 모두 private
3. price의 getter는 public, setter는 private

## 4. 자바와 코틀린을 함께 사용할 경우 주의점
---
- 코틀린과 자바의 protected는 서로 다르다
- 자바는 같은 패키지의 코틀린 protected에 접근할 수 있다