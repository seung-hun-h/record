## 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라
---
- 열거 타입은 열거한 값들을 그대로 가져와서 다음 값을 더 추가하여 다른 목적으로 사용할 수 없다
- 이러한 설계는 실수가 아니다. 대부분의 경우 열거 타입을 확장하는 것은 좋지 않은 생각이다
- 그러나 확장할 수 있는 열거 타입이 어울리는 쓰임이 최소한 하나는 있다. 바로 연산 코드다
	- 이따금 API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가할 수 있도록 열어줘야 할 때가 있다
- 열거 타입으로 이 효과를 내는 방법은 바로 인터페이스를 구현하는 것이다

```Java
public enum BasicOperation implements Operation {  
   PLUS("+") {  
      @Override  
      public double apply(double x, double y) {  
         return x + y;  
      }  
   },  
   MINUS("") {  
      @Override  
      public double apply(double x, double y) {  
         return x - y;  
      }  
   },  
   TIMES("*") {  
      @Override  
      public double apply(double x, double y) {  
         return  x * y ;  
      }  
   },  
   DIVIDE("/") {  
      @Override  
      public double apply(double x, double y) {  
         return x / y;  
      }  
   };  
  
   private final String symbol;  
  
   BasicOperation(String symbol) {  
      this.symbol = symbol;  
   }  
  
   @Override  
   public String toString() {  
      return symbol;  
   }  
}
```

```Java
package item38;  
  
import java.util.Arrays;  
import java.util.Collection;  
import java.util.List;  
  
public enum ExtendedOperation implements Operation{  
   EXP("^") {  
      @Override  
      public double apply(double x, double y) {  
         return Math.pow(x, y);  
      }  
   },  
   REMAINDER("%") {  
      @Override  
      public double apply(double x, double y) {  
         return x % y;  
      }  
   };  
  
   private final String symbol;  
  
   ExtendedOperation(String symbol) {  
      this.symbol = symbol;  
   }  
  
   @Override  
   public String toString() {  
      return symbol;  
   }  
}
```

- 개별 인스턴스 수준뿐 아니라 타입 수준에서도 기본 열거 타입 대신 확장된 열거 타입을 넘겨 확장된 열거 타입의 원소 모두를 사용할 수 있게 할 수도 있다
```Java
public static void main(String[] args) {  
   double x = 1.4;  
   double y = 2.1;  
   test(ExtendedOperation.class, x, y);  
}  
  
private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {  
   for (Operation op : opEnumType.getEnumConstants()) {  
      System.out.println("op.apply(x, y) = " + op.apply(x, y));  
   }  
}
```

- `test` 메서드에 `ExtendedOperation` 클래스 리터럴을 전달해 확장된 연산자들이 무엇인지 알려준다
- `<T extends Enum<T> & Operation>`는 Class 객체가 열거 타입인 동시에 `Operation`의 하위 타입이어야 한다는 뜻이다

- 또 다른 대안은 Class 객체 대신 한정적 와일드카드 타입인 `Collection<? extends Operation>`을 넘겨주는 방법이다

```Java
public static void main(String[] args) {  
   double x = 1.4;  
   double y = 2.1;  
   test(Arrays.asList(ExtendedOperation.values()), x, y);  
}  
  
private static void test(Collection<? extends Operation> opSet, double x, double y) {  
   for (Operation operation : opSet) {  
      System.out.println("operation.apply(x, y) = " + operation.apply(x, y));  
   }  
}
```

- 인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식에도 한 가지 문제점이있다
	- 열거 타입끼리 구현을 상속할 수 없다는 점이다
- 아무 상태에도 의존하지 않는 경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방법이있다
- `Operation`에는 연산 기호를 저장하고 찾는 로직이 두 구현체에 모두 들어가야 한다
- 이 경우에는 중복량이 적으니 문제가 되지 않지만, 공유하는 기능이 많은 경우 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 중복을 해결할 수 있을 것이다
