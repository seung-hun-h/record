## ITEM23 태그 달린 클래스보다는 클래스 계층 구조를 활용하라

>
 태그 달린 클래스를 써야 하는 상황은 거의 없다. 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층구조로 대체하는 방법을 생각해보자. 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩터링하는 걸 고민해보자

- 다음은 원과 사각형을 표현할 수 있는 클래스이다

```Java
public class Figure {  
   enum Shape {RECTANGLE, CIRCLE,}  
  
   // 태그 필드 - 현재 모양을 나타낸다  
   final Shape shape;  
  
   // 모양이 사각형일 때만 쓰인다  
   double length;  
   double width;  
  
   // 모양이 원일 떄만 쓰인다  
   double radius;  
  
   // 원용 생성자  
   public Figure(double radius) {  
      this.shape = Shape.CIRCLE;  
      this.radius = radius;  
   }  
  
   // 사각형용 생성자  
   public Figure(double length, double width) {  
      this.shape = Shape.RECTANGLE;  
      this.length = length;  
      this.width = width;  
   }  
  
   double area() {  
      return switch (shape) {  
         case CIRCLE -> Math.PI * (radius * radius);  
         case RECTANGLE -> length * width;  
      };  
   }  
}
```

#### 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다
- 열거타입, 태그 필드, switch 문 등 쓸데 없는 코드가 많다
- 여러 구현이 한 클래스에 혼합돼 가독성도 나쁘다
- 다른 의미를 위한 코드가 항상 함께 존재하니 메모리도 많이 사용한다
- 필드들을 final로 선언하려면 쓰이지 않는 필드들까지 생성자에서 초기화 해야 한다
- 생성자가 태그 필드를 설정하고 해당 의미에 쓰이는 데이터 필드들을 초기화하는 데 컴파일러가 도와줄 수 있는 것은 별로 없다
- 엉뚱한 필드를 초기화 해도 런타임에 드러난다
- 또 다른 의미를 추가하려면 코드를 수정해야 한다

### 태그 클래스를 계층구조로 변경
1. 루트가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다
2. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다
3. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다
4. 루트 클래스가 정의한 추상 메서드를 각자의 의미에 맞게 구현한다

```Java
public abstract class Figure {  
   abstract double area();  
}
```

```Java
public class Rectangle extends Figure {  
   final double length;  
   final double width;  
  
   public Rectangle(double length, double width) {  
      this.length = length;  
      this.width = width;  
   }  
  
   @Override  
   double area() {  
      return length * width;  
   }  
}
```

```Java
public class Circle extends Figure {  
   final double radius;  
  
   public Circle(double radius) {  
      this.radius = radius;  
   }  
  
   @Override  
   double area() {  
      return Math.PI * (radius * radius);  
   }  
}
```

#### 계층 구조의 장점
- 간결하고 명확하다
- 각 의미를 독립된 클래스로 분리하여 관렵 없던 데이터 필드가 사라졌다
- 각 클래스의 필드를 final로 선언할 수 있다
- 각 클래스의 생성자가 모든 필드를 초기화 했는지, 추상 메서드를 구현했는지 컴파일러가 검사해준다
- switch 문이 사라졌다
- 프로그래머들이 독립적으로 계층구조를 확장하고 사용할 수 있다
- 변수의 의미를 명시하거나 제한할 수 있고, 특정 의미만 매개 변수로 받을 수 있다
