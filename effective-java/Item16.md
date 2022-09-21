## ITEM16 public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

>
>public 클래스는 절대 가변 필드를 직접 노출해서는 안된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-private 클래스 혹은 private 중첩 클래스에서는 종종 필드를 노출하는 편이 나을 떄도 있다
>

- 캡슐화의 이점
	- API를 수정하지 않고 내부 표현을 변경할 수 있다
	- 불변식을 보장할 수 있다
	- 외부에서 API를 호출할 때 부수적인 작업을 할 수 있다
- 필드를 public으로 공개한 클래스는 캡슐화의 이점을 누릴 수 없다
- 객체 지향 프로그래머는 필드를 공개하지 않기 위해 접근자, 수정자를 제공한다
```Java
public class Point {
	private double x;
	private double y;

	public Point(double x, double y) {
		this.x = x;
		this.y = y;
	}

	public double getX() { return x; }
	public double getY() { return y; }
	public void setX(double x) {
		this.x = x;
	}
	public void setY(double y) {
		this.y = y;
	}
	
}
```

- 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다
- 하지만 package-private 혹은 private 중첩클래스라면 데이터 필드를 노출한다해도 하등 문제가 없다

### public 클래스의 필드가 불변인 경우
- public 클래스의 필드가 불변인 경우 직접 노출할 때의 단점은 조금 줄어든다
	- 불변식을 보장할 수 있다
- 여전히 API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다