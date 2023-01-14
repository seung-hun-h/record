## 1. 중첩 클래스의 종류
---
- 자바를 기준으로 중첩 클래스의 종류를 나누면 다음과 같다
	- static을 사용하는 중첩 클래스
	- static을 사용하지 않는 중첩 클래스
		- 내부 클래스
		- 지역 클래스
		- 익명 클래스
- static 을 붙인 클래스는 바깥 클래스를 직접 참조할 수 없다
- 그렇지 않은 경우 `바깥 클래스 타입.this.대상` 형식으로 바깥 클래스를 참조할 수 있다

```Java
public class JavaHouse {
	private String address;
	private LivingRoom livingRoom;

	public LivingRoom(String address) {
		this.address = address;
		this.livingRoom = new LivingRoom(10);
	}

	public LivingRoom getLivingRoom() {
		return livingRoom;
	}

	public class LivingRoom {
		private double area;

		public LivingRoom(double area) {
			this.area = area;
		}

		public String getAddress() {
			return JavaHouse.this.address;
		}
	}
}
```

```Java
public static void main(String[] args) {
	JavaHouse house = new JavaHouse("제주도");
	System.out.println(house.getLivingRoom().getAddress());
}
```

### 내부 클래스의 단점
1. 내부 클래스는 숨겨진 외부 클래스 정보를 가지고 있어 참조를 해지하지 못하는 경우 메모리 누수가 생길 수 있고 디버깅하기 어렵다
2. 내부 클래스의 직렬화 형태가 명확하기 정의되어 있지 않아 직렬화에 제한이있다

- 따라서 중첩 클래스를 만들때는 `static` 클래스를 사용하라

## 2. 코틀린의 중첩 클래스와 내부 클래스
---
### 코틀린 중첩 클래스
```Kotlin
class House (
	private val address: String,
	private val livingRoom: LivingRoom,
) {
	class LivingRoom ( // 1
		private val area: Double
	)
}
```
1. 코틀린은 기본적으로 자바의 `static` 중첩 클래스이다

### 코틀린 내부 클래스
```Kotlin
class House (
	private val address: String,
	private val livingRoom: LivingRoom,
) {
	inner class LivingRoom ( // 1
		private val area: Double
	) {
		val address: String
			get() = this@House.address // 2
	}
}
```

1. `inner` 키워드를 사용해 내부 클래스를 선언한다
2. `this@바깥클래스타입` 형식으로 바깥 클래스를 직접 참조한다

- 코틀린은 기본적으로 바깥 클래스를 참조하지 않는다. 바깥 클래스를 참조하고 싶으면 inner 키워드를 사용해야 한다