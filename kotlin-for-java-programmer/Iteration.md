## 1. for-each
---
### 자바의 for-each
```Java
List<Long> numbers = Arrays.asList(1L, 2L, 3L);
for (long number : numbers) {
	System.out.println(number);
}
```

### 코틀린의 for-each
```Kotlin
val numbers = listOf(1L, 2L, 3L)
for (number in numbers) {
	println(number)
}
```
- `in`을 사용한다
- `in` 뒤에는 `iterable`을 구현한 타입이 위치한다

## 2. 전통적인 for문
---
### 자바의 for 문
```Java
for (int i = 1; i <= 3; i++) { // 1에서 3까지 1씩 증가
	System.out.println(i);
}

for (int i = 3; i >= 1; i--) { // 3에서 1까지 1씩 감소
	System.out.println(i);
}

for (int i = 1; i <= 5; i += 2) { // 1에서 5까지 2씩 증가
	System.out.println(i);
}
```

### 코틀린의 for문
```Kotlin
for (i in 1..3) { // 1에서 3까지 1씩 증가
	println(i)
}

for (i in 3 downTo 1) { // 3에서 1까지 1씩 감소
	println(i)
}

for (i in 1..5 step 2) { // 1에서 5까지 2씩 증가
	println(i)
}
```
- `..` 는 범위를 만들어 내는 연산자이다
	- `IntRange` -> `IntProgression`
	- Progression은 등차수열이다. `IntProgression`은 시작 값, 끝 값, 공차를 받는 객체
- `downTo`, `step` 는 중위 함수이다
	- `변수.함수이름(argument)` 대신 `변수 함수이름 argument`

## 4. while 문
### 자바의 while 문
```Java
int i = 1;
while (i <= 3) {
	System.out.println(i++);
}
```

### 코틀린의 while문
```Kotlin
var i = 1
while(i <= 3) {
	println(i++)
}
```

- `while`, `do-while`은 자바와 사용방법이 동일하다