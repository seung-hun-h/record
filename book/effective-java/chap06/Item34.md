## int 상수 대신 열거 타입을 사용하라
---
- 열거 타입은 일저 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다
- 열거 타입을 사용하기 전에는 정수 상수를 한 묶음으로 선언해서 사용하곤 했다

### 정수 상수
```Java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```
- 정수 열거 패턴의 단점
	- 타입 안전을 보장할 수 없고 표현력도 좋지 않다
	- 정수 열거 패턴을 위한 별도의 이름공간(namespace)를 제공하지 않는다
	- 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다. 따라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야한다
- 문자열 열거 패턴도 존재하는데 이건 더 나쁘다
	- 하드 코딩한 문자열에 오타가 있어도 컴파일러는 확인할 길이 없으니 자연스럽게 런타임 버그가 생긴다

### 열거 타입
```Java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

- 자바의 열거타입은 완전한 형태의 클래스다
- 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final`필드로 공개한다
	- 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다
- 열거 타입으로 만들어진 인스턴스는 딱 하나만 존재한다


### 열거 타입의 특징
- 열거 타입은 완전한 형태의 클래스다
- 열거 타입은 싱글턴을 일반화한 형태다
- 컴파일타임 타입 안전성을 보장한다
- 각자의 이름공간이 있어 이름이 같은 상수도 평화롭게 공존한다
- 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 하지 않아도 된다
- `toString`메서드는 출력하기에 적합한 문자열을 내어준다
- 메서드나 필드를 추가할 수 있고 인터페이스를 구현할 수도 있다
	- Object 메서드를 높은 품질로 구현했다
	- Comparable, Serializable을 구현했다
	- 직렬화 형태도 왠만큼 변형을 가해도 문제없이 동작하게끔 구현해놨다

### 데이터와 메서드를 갖는 열거 타입
```Java
public enum Planet {  
   MERCURY(3.302e+23, 2.439e6),  
   VENUS  (4.869e+24, 6.052e6),  
   EARTH  (5.975e+24, 6.378e6),  
   MARS   (6.419e+23, 3.393e6),  
   JUPITER(1.899e+27, 7.149e7),  
   SATURN (5.685e+26, 6.027e7),  
   URANUS (8.683e+25, 2.556e7),  
   NEPTUNE(1.024e+26, 2.477e7);  
  
   private final double mass;             
   private final double radius;           
   private final double surfaceGravity;   
   private static final double G = 6.67300E-11;  
  
   Planet(double mass, double radius) {  
      this.mass = mass;  
      this.radius = radius;  
      surfaceGravity = G * mass / (radius * radius);  
   }  
  
   public double mass()           { return mass; }  
   public double radius()         { return radius; }  
   public double surfaceGravity() { return surfaceGravity; }  
  
   public double surfaceWeight(double mass) {  
      return mass * surfaceGravity;  
   }  
}
```
- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저정하면 된다
- 열거 타입은 근본적으로 불변이라 모든 필드가 `final`이어야 한다
- 열거 타입을 제거하면 제거된 코드를 사용하는 클라이이언트가 컴파일을 다시하면 컴파일 오류가 발생한다

### 열거 타입을 사용하는 팁
- 열거 타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 private이나 package-private 메서드로 구현한다
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱 레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만든다

### 값에 따라 분기하는 열거 타입 해결
```Java
public enum Operation {  
   PLUS, MINUS, TIMES, DIVIDE;  
  
   public double apply(double x, double y) {  
      switch (this) {  
         case PLUS -> {  
            return x + y;  
         }  
         case MINUS -> {  
            return x - y;  
         }  
         case TIMES -> {  
            return x * y;  
         }  
         case DIVIDE -> {  
            return x / y;  
         }  
         default -> throw new AssertionError("지원하지 않는 연산입니다." + this);  
      }  
   }  
}
```
- switch문을 사용하면 유지 보수에 좋지않다
	- 깜빡하고 뺴먹으면 `AssertionError`예외가 발생한다
	- 새로운 연산을 추가하려면 `case`문을 추가해야 한다

#### 상수별 메서드 구현
```Java
public enum Operation1 {  
   PLUS("+") {  
      @Override  
      public double apply(double x, double y) {  
         return x + y;  
      }  
   },  
   MINUS("-") {  
      @Override  
      public double apply(double x, double y) {  
         return x - y;  
      }  
   },  
   TIMES("*") {  
      @Override  
      public double apply(double x, double y) {  
         return x * y;  
      }  
   },  
   DIVIDE("/") {  
      @Override  
      public double apply(double x, double y) {  
         return x / y;  
      }  
   };  
  
   public abstract double apply(double x, double y);  
  
   private static final Map<String, Operation1> stringToEnum = Stream.of(values()).collect(Collectors.toMap(Objects::toString, e -> e));  
  
   private final String symbol;  
  
   Operation1(String symbol) {  
      this.symbol = symbol;  
   }  
  
   @Override  
   public String toString() {  
      return symbol;  
   }  
  
   public static Optional<Operation1> fromString(String symbol) {  
      return Optional.ofNullable(stringToEnum.get(symbol));  
   }  
}
```
- 새로운 상수를 추가할 때 `apply`도 재정의 해야하는 것을 쉽게 알 수 있다
	- 재정의 하지 않았다면 컴파일 오류가 발생한다
- 열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 `valueOf(String)`가 자동 생성된다
- 열거 타입의 `toString`을 재정의 하려거든, `toString`이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 `fromString` 메서드도 함께 제공하는 것을 고려해보자

```Java
private static final Map<String, Operation1> stringToEnum = Stream.of(values()).collect(Collectors.toMap(Objects::toString, e -> e));  

public static Optional<Operation1> fromString(String symbol) {  
  return Optional.ofNullable(stringToEnum.get(symbol));  
}  
```
- 자바 8 이전에는 빈 해시맵을 만든 다음 `values`가 반환한 배열을 순회하며 {문자열, 열거 타입 상수} 쌍을 맵에 추가했을 것이다
- 지금도 이렇게 구현해도 된다. 하지만 열거 타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없다
- 열거 타입의 정적 필드 중 열거 타입의 생성자에서 접근할 수 있는 것은 상수 변수뿐이다
- 열거 타입 생성자가 실행되는 시점에는 정적 필드들이 아직 초기화되기 전이라 자기 자신을 추가하지 못하게하는 제약이 꼭 필요하다(?)
	- 일반 클래스는 이렇지 않고, Enum만 이러한 것 같다

### 기존 열거 타입에 상수별 동작을 혼합하는 경우
- 기존 열거 타입에 상수별 동작을 혼합해 넣는 경우에는 switch 문이 좋은 선택일 수 있다
```Java
public static Operation inverse(Operation op) {
	switch(op) {
		case PLUS: return Operation.MINUS;
		case MINUS: return Operation.PLUS;
		case TIMES: return Operation.DIVIDE;
		case DIVIDE: return Operation.TIMES;
		default: throw new AssertionError("알 수 없는 연산" + op);
	}
}
```


### 열거 타입은 언제쓸까?
- 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거타입을 사용한다
- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다