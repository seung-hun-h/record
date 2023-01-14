## 1. 배열
---
- 배열은 잘 사용하지 않는다. 
- 리스트 사용을 권장한다

```Kotlin
val array = arrayOf(100, 200) // 1

for (i in array.indices) { // 2
	println("$i $array[i]")
}

for ((idx, value) in array.withIndex()) { // 3
	println("$idx $value")
}

for (value in array) { // 4
	println("$value")
}

array.plus(300) // 5
```

1. `arrayOf` 정적 팩토리 메서드로 배열을 생성한다
2. `indices`로 인덱스를 조회한다
3. `withIndex()`로 인덱스와 값을 함께 조회한다
4. for-each 문으로 값을 조회한다
5. `plus()`로 배열에 값을 추가한다


## 2. Collection - List, Set, Map
---
- 컬렉션을 만들때 가변인지 불변인지 먼저 정해야 한다
- 코틀린의 컬렉션에는 기본적으로 List, Set, Map이 있고 이들은 불변이다
- 각 컬렉션의 가변 타입이 있는데 각각 MutableList, MutableSet, MutableMap 이다

### List
```Kotlin
val numbers = listOf(100, 200) 
val emptyNumbers = emptyList<Int>() // 1

numbers.get(0)
numbers[0] // 2

for (number in numbers) {
...
}

for((idx, number) in numbers.withIndex()) {
...
}
```
1. 빈 리스트를 만들때는 타입을 추론할 수 없으므로 원소 타입을 명시한다
2. 배열처럼 인덱스로 바로 접근할 수 있다
- MutableList의 기본 구현체는 ArrayList이다

### Set
- List와 달리 순서가 없고, 원소의 중복을 허용하지 않는다
- 자료구조적 의미만 제외하면 List와 동일하다
- MutableSet의 기본 구현체는 LinkedHashSet이다

### Map
```Kotlin
val map = mutableMapOf<Int, String>()
map[1] = "MONDAY"
map[2] = "TUESDAY"

mapOf(1 to "MONDAY", 2 to "TUESDAY")

for (key in map.keys) {
...
}

for ((key, value) in map.entries) {
...
}
```




## 3. 컬렉션의 null 가능성, Java와 함께 사용하기
---
### null 가능성
- `List<Int?>` : 리스트에 null이 들어갈 수 있지만, 리스트는 null이 아님
- `List<Int>?`: 리스트에 null이 들어갈 수 없지만, 리스트는 null이 될 수 있음
- `List<Int?>?`: 리스트에 null이 들어갈 수 있고, 리스트도 null이 될 수 있음

### 자바는 읽기 전용과 변경 가능 컬렉션을 구분하지 않는다
- 코틀린 쪽의 컬렉션이 자바에서 호출되면 컬렉션 내용이 변할 수 있음을 감안해야 한다
- 코틀린 쪽에서 Collections.unmodifiableXXX()을 활용하면 변경을 막을 수 있다
- 코틀린에서 자바의 컬렉션을 가져다 사용할 떄는 플랫폼 타입을 신경써야 한다
	- `List<Int?>`, `List<Int>?`, `List<Int?>?` 중 어느것인지 모른다
	- 맥락을 확인하고 자바 코드를 가져오는 지점을 wrapping 해야 한다

