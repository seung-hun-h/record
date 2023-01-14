## 1. Data Class
---
- DTO(Data Transfer Object)는 계층간의 데이터를 전달하기 위한 객체이다
	- 데이터(필드)
	- 생성자
	- getter
	- equals, hashCode
	- toString
	- 같은 것들을 가지고 있다

### 코틀린 - Data Class
- `data class 클래스명` 으로 손쉽게 DTO를 만들 수 있다
	- 자바에서 직접 구현해야 했던 것들을 직접 구현하지 않아도 된다
	- named argument를 활용하면 builder를 사용하는 효과를 얻을 수도 있다
```Kotlin
data class PersonDto (
	val name: String,
	val age: Int
)
```

## 2. Enum Class
---
- Enum 클래스는 다음과 같은 특징을 가지고있다
	- 추가적은 클래스를 상속받을 수 없다
	- 인터페이스는 구현 가능하다
	- 싱글톤이다

### 코틀린 Enum
```Kotlin
enum class Country (
	val code: String
) {
	KOREA("KO"),
	AMERICA("US"),
}
```
- 자바의 Enum과 문법상 차이가 거의 없다

## 3. Sealed Class, Sealed Interface
---
- 상속 가능하도록 추상 클래스를 만들고 싶은데, 외부에서는 이 클래스를 상속받지 않게 할때 사용한다
- 특징
	- 컴파일 타임에 하위 클래스 타입을 모두 기억한다
	- 런타임에 클래스 타입이 추가될 수 없다
- Enum과 차이
	- 클래스를 상속받을 수 있다
	- 하위 클래스는 멀티 인스턴스가 가능하다

```Kotlin
sealed class HyundaiCar (
	val name: String, 
	val price: Long
)

class Avante : HyundaiCar("아반떼", 1_000L)

class Sonata : HyundaiCar("소나타", 2_000L)

class Grandeur : HyundaiCar("그랜져", 3_000L)
```
- 컴파일 타임에 하위 타입을 모두 기억하는 특징 덕분에 `when`절에서 유용하게 사용될 수 있다
- 추상화가 필요한 Entity, DTO에 Sealed Class를 활용할 수 있다
