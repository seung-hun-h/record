
> 
> 꼭 필요한 경우가 아니면 equals를 재정의 하지 말자. 많은 경우에 Object의 equals가 여러분이 원하는 비교를 정확히 수행해준다. 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다
> 

- `equals`메서드는 재정의하기 쉬워 보이지만, 곳곳에 함정이 도사리고 있다
- 그냥 두면 그 클래스의 인스턴스는 오직 자기 자신과만 같게된다

### equals를 재정의 하지 말아야하는 상황들
**1. 각 인스턴스가 본질적으로 고유하다**
- 값을 표현하는 것이 아니라 동작하는 개체를 표현하는 클래스가 여기에 해당한다
**2. 인스턴스의 논리적 동치성을 검사할 일이 없다**
**3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 맞다**
- Set의 구현체는 AbstractSet, List의 구현체는 AbstractList, Map의 구현체는 AbstractMap으로 부터 구현된 equals를 그대로 쓴다
**4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다**
- equals가 실수라도 호출되는 것을 막고 싶다면 다음과 같이 구현한다
```Java
@Override
public boolean equals(Object o) {
	throw AssertionError(); // 호출 금지
}
```

### equals는 언제 재정의 해야 할까?
- 논리적 동치성을 확인해야 하는데 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 하지 않았을 때다
- 주로 값 클래스들이 여기에 해당한다
- 단, 값 클래스라도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 equals를 재정의 하지 않아도 된다
	- Enum이 여기에 해당한다
	- 이런 클래스는 논리적 동치성과 객체 식별성이 사실상 똑같은 의미다

### equals 재정의 규약
- 동치관계: 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산
- 동치류: 집합을 서로 같은 원소들로 이루어진 부분집합으로 나눌 때 해당 부분집합
- equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다
- 재정의 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다

**1. 반사성**
- 객체는 자기 자신과 같아야 한다
**2. 대칭성**
- 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다
```Java
public final class CaseInsensitiveString {  
   private final String s;  
  
   public CaseInsensitiveString(String s) {  
      this.s = Objects.requireNonNull(s);  
   }  
  
   // 대칭성 위배  
   @Override  
   public boolean equals(Object obj) {  
      if (obj instanceof CaseInsensitiveString) {  
         return s.equalsIgnoreCase(((CaseInsensitiveString)obj).s);  
      }  
  
      if (obj instanceof String) {  
         return s.equalsIgnoreCase((String)obj);  
      }  
  
      return false;  
   }  
}
```
- 위 코드는 대칭성을 위배한다
```Java
public static void main(String[] args) {  
   CaseInsensitiveString cis = new CaseInsensitiveString("Polish");  
   String str = "polish";  
  
   System.out.println("cis.equals(str) = " + cis.equals(str)); // true  
   System.out.println("str.equals(cis) = " + str.equals(cis)); // false  
}
```
- 문제를 해결하기 위해서는 깔끔하게 String과 연동하겠다는 의도를 제거하면된다
```Java
@Override  
public boolean equals(Object obj) {  
   return obj instanceof CaseInsensitiveString  
      && ((CaseInsensitiveString)obj).s.equals(s);  
}
```

**3. 추이성**
- 첫 번째와 두 번째 객체가 같고, 두 번째와 세 번째 객체가 같을 때, 첫 번째와 세 번째 객체도 같아야 한다
```Java
public class Point {  
   private final int x;  
   private final int y;  
  
   public Point(int x, int y) {  
      this.x = x;  
      this.y = y;  
   }  
  
   @Override  
   public boolean equals(Object obj) {  
      if (!(obj instanceof Point p)) {  
         return false;  
      }  
      return p.x == this.x && p.y == this.y;  
   }  
}
```

```Java
public class ColorPoint extends Point {  
   private final Color color;  
  
   public ColorPoint(int x, int y, Color color) {  
      super(x, y);  
      this.color = color;  
   }  
}
```

- 상위 클래스인 Point의 equals를 그대로 사용하면 ColorPoint는 색상 정보를 무시한채 비교하게 된다
- 색상 정보는 필수 정보이므로 equals를 재정의 하는 것이 좋다

```Java
@Override  
public boolean equals(Object obj) {  
   if (!(obj instanceof ColorPoint p)) {  
      return false;  
   }  
   return super.equals(obj) && p.color == this.color;  
}
```

- 이 메서드는 일반 Point를 ColorPoint에 비교한 결과와 그 반대가 다른 값을 반환한다. 즉, 대칭성을 위배한다

```Java
public static void main(String[] args) {  
   Point p = new Point(1, 2);  
   ColorPoint cp = new ColorPoint(1, 2, Color.RED);  
  
   System.out.println("p.equals(cp) = " + p.equals(cp)); // true
   System.out.println("cp.equals(p1) = " + cp.equals(p)); // false
}
```

- 대칭성을 지키기 위해 Point와 비교할 때는 색상을 무시하도록 재정의 할 수 있다

```Java
@Override  
public boolean equals(Object obj) {  
   if (!(obj instanceof Point)) {  
      return false;  
   }  
     
   // obj가 일반 Point면 색상을 무시하고 비교한다  
   if (!(obj instanceof ColorPoint p)) {  
      return obj.equals(this);  
   }  
     
   return super.equals(obj) && p.color == this.color;  
}
```

- 하지만 이 방식은 추이성을 위배한다

```Java
public static void main(String[] args) {  
	ColorPoint p1 = new ColorPoint(1, 2, Color.RED);  
	Point p2 = new Point(1, 2);  
	ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);  
  
	System.out.println("p1.equals(p2) = " + p1.equals(p2)); // true
	System.out.println("p2.equals(p3) = " + p2.equals(p3)); // true
	System.out.println("p3.equals(p1) = " + p3.equals(p1)); // false
}
```

- 그리고 이 방식은 추이성을 위배할 뿐 아니라 무한 재귀에 빠질 위험도 있다
	- SmellPoint를 만들고 equals를 같은 방식으로 구현한 후, `myColorPoint.equals(mySmelloPoint)`를 호출하면 StackOverflowError가 발생한다
- 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다
- instanceof 검사를 `getClass()` 검사로 바꿀 수 있다고 생각하겠지만 이는 리스코프 치환 원칙을 위배한다
	- 상위 클래스와 하위 클래스의 getClass() 값이 다를 수 있기 때문이다
	- Point의 하위 클래스는 정의상 여전히 Point이므로 어디서든 Point로써 활용될 수 있어야 한다
- 이러한 문제를 해결하기 위한 우회적인 방법으로 '상속 대신 컴포지션을 활용하라'라는 아이템18의 조언을 따르면 된다

```Java
public class ColorPoint {  
   private final Point point;  
   private final Color color;  
  
   public ColorPoint(Point point, Color color) {  
      this.point = point;  
      this.color = color;  
   }  
  
   @Override  
   public boolean equals(Object obj) {  
      if (!(obj instanceof ColorPoint cp)) {  
         return false;  
      }  
      return cp.point.equals(point) && cp.color == color;  
   }  
}
```

> 추상 클래스의 하위 클래스에서라면 equals 규약을 지키면서도 값을 추가할 수 있다.
> 상위 클래스를 직접 인스턴스로 만드는 게 불가능하다면 지금까지 이야기한 문제는 일어나지 않는다.

**4. 일관성**
- 두 객체가 같다면, 두 객체가 수정되지 않는 한 앞으로도 영원히 같아야 한다
- 클래스를 작성할 때는 불변 클래스로 만드는 게 나을지를 심사숙고하자
- 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다
	- equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다
	  
**5. null-아님**
- 모든 객체가 null과 같지 않아야 한다
- `obj.equals(null)`이 true를 반환한다던가, NullpointerException을 던지지 않아야 한다
- 그리고 아래 처럼 명시적으로 null을 검사할 필요도 없다

```Java
@Override
public boolean equals(Object obj) {
	if (obj == null) {
		return false;
	}
	...
}
```

- 객체의 필드를 확인하기 위해 형변환을 해야 하는데, 이를 위해 instanceof를 사용하게 된다
- instanceof는 첫 번째 인자가 null이면 false를 반환한다

```Java
@Override
public boolean equals(Object obj) {
	if (!(obj instanceof MyType)) {
		return false;
	}
	...
}
```

### 양질의 equals 메서드 구현법
1. `==`  연산자를 사용해 입력이 자기 자신의 참조인지 확인한다
2. `instanceof` 연산자로 입력이 올바른 타입인지 확인한다
3. 입력을 올바른 타입으로 형변환한다
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다

- float, double을 제외한 기본 타입 필드는 == 연산자를 사용해 비교한다
- float, double은 Float.NaN, -0.0f 처럼 특수한 값 때문에 `Float.compare()`, `Double.compare()`를 사용한다
	- Float.equals(), Double.equals()는 오토박싱으로 인해 성능 저하가 발생할 수 있다
- 배열의 모든 요소가 핵심 필드라면 `Arrays.equals()`를 사용한다
- null도 정상 값으로 취급하는 참조 타입 필드도 있다. 이 경우에는 `Objects.equals()`를 사용하자
	- NullPointerException을 방지한다
- 비교하기가 아주 복잡한 필드를 가진 클래스는 그 필드의 표준형을 저장해둔 후 표준형끼리 비교하면 훨씬 경제적이다
- 성능에 민감하다면 다를 가능성이 크거나 비교하는 비용이 싼 필드를 먼저 비교하자
	- 동기화용 락처럼 객체의 논리적 상태와 관련 없는 필드는 비교할 필요가 없다
	- 파생 필드가 객체 전체 상태를 대표하는 상황에서는 파생 필드를 비교하는 것이 더 빠를 때도 있다

### 주의 사항
- equals를 재정의할 땐 hashCode도 반드시 재정의하자
- 너무 복잡하게 해결하려 들지 말자
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자
	- Object 외의 타입을 매개변수로 받는 경우 Object.equals를 재정의한 것이 아니라 다중 정의한 것이다
	-  Object 외의 타입을 매개변수로 받는 경우 거짓 양성을 내게하고 보안 측면에서도 잘못된 정보를 준다
	- `@Override` 애너테이션을 사용하면 Object 외 타입을 매개 변수로 받는 경우 컴파일이 되지 않는다