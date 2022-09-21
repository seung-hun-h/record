## ITEM17 변경 가능성을 최소화하라
### 불변 클래스란?
- 불변 클래스란 간단히 그 인스턴스의 내부 값을 수정할 수 없는 클래스를 말한다
- 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 위우며, 오류가 생길 여지도 적고 훨씬 안전하다

#### 불변 클래스를 만들기위 한 규칙
1. 객체의 상태를 변경하는 메서드를 제공하지 않는다
2. 클래스를 확장할 수 없도록 한다
3. 모든 필드를 final로 선언한다
4. 모든 필드를 private으로 선언한다
5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다

```Java
public final class Complex {  
   private final double re;  
   private final double im;  
  
   public Complex(double re, double im) {  
      this.re = re;  
      this.im = im;  
   }  
  
   public double realPart() {  
      return re;  
   }  
  
   public double imaginaryPart() {  
      return im;  
   }  
  
   public Complex plus(Complex c) {  
      return new Complex(re + c.re, im + c.im);  
   }  
  
   public Complex minus(Complex c) {  
      return new Complex(re - c.re, im - c.im);  
   }  
  
   public Complex times(Complex c) {  
      return new Complex(re * c.re - im * c.im, re * c.re + im * c.im);  
   }  
  
   public Complex dividedBy(Complex c) {  
      double tmp = c.re * c.re + c.im * c.im;  
      return new Complex((re * c.re + im * c.im) / tmp,  
                     (re * c.re - im * c.im) / tmp);  
   }  
  
   @Override  
   public boolean equals(Object o) {  
      if (this == o) {  
         return true;  
      }  
      if (o == null || getClass() != o.getClass()) {  
         return false;  
      }  
  
      Complex complex = (Complex)o;  
  
      if (Double.compare(complex.re, re) != 0) {  
         return false;  
      }  
      return Double.compare(complex.im, im) == 0;  
   }  
  
   @Override  
   public int hashCode() {  
      int result;  
      long temp;  
      temp = Double.doubleToLongBits(re);  
      result = (int)(temp ^ (temp >>> 32));  
      temp = Double.doubleToLongBits(im);  
      result = 31 * result + (int)(temp ^ (temp >>> 32));  
      return result;  
   }  
  
   @Override  
   public String toString() {  
      return "Complex{" +  
         "re=" + re +  
         ", im=" + im +  
         '}';  
   }  
}
```

- Complex는 실수부와 허수부로 구성된 수인 복소수를 표현한다
	- 접근자 메서드와 사칙 연산 메서드를 가진다
- 사칙 연산 메서드가 필드 값을 변경하지 않고 새로운 Complex 인스턴스를 반환하도록 되어 있다
	- 이렇게 피연산자로 사용된 인스턴스는 그대로 두고 새로운 인스턴스를 반환하도록 하는 프로그래밍 기법을 함수형 프로그래밍이라 한다
	- 메서드 이름도 동사(add)가 아닌 전치사(plus)를 사용했다
- 함수형 프로그래밍 기법을 사용하면 불변이되는 영역의 비율이 높아지는 장점을 누릴 수 있다

### 불변 객체의 장점
#### 불변 객체는 단순하다
- 불변 객체는 생성된 시점의 상태가 파괴될 때까지 그대로 간직한다
- 가변 클래스는 믿고 사용하기 어려울 수 있다

#### 불변 객체는 스레드 안전하다
- 불변 객체는 여러 스레드가 동시에 사용해도 절대 훼손되지 않는다
- 불변 객체는 스레드 안전하므로 안심하고 공유할 수 있다
- 자주 사용되는 인스턴스를 캐싱하여 인스턴스를 중복 생성하지 않도록 정적 팩터리를 제공할 수 있다
	- public 생성자 대신 정적 팩터리를 제공하면 나중에 캐싱을 덧붙일 수 있다
- 불변 클래스는 clone 메서드나 복사 생성자를 제공하지 않는게 좋다.
	- 아무리 복사해봐야 원본과 동일하다

#### 불변 객체 내부 데이터를 공유할 수 있다
-  BigInteger 클래스는 내부에서 값의 부호와 크기를 따로 표현한다
	- 부호에는 int, 크기에는 int 배열을 사용한다
- negate 메서드는 크기가 같고 부호만 반대인 새로운 BigInteger를 생성하는데, 이 때 배열은 비록 가변이지만 복사하지 않고 원본 인스턴스와 공유해도 된다

#### 불변 객체를 다른 불변 객체의 구성요소로 사용하면 이점이 많다
- 불변 객체로 이루어진 객체라면 구조가 복잡하더라도 불변식을 유지하기 훨씬 수월하다

#### 불변 객체는 그 자체로 실패 원자성을 제공한다
- 상태가 변하지 않으므로 잠깐이라도 불일치 상태에 빠질 일이 없다

### 불변 객체의 단점
- 값이 다르면 반드시 독립된 객체로 만들어야 한다
- 원하는 객체를 완성하기까지의 단계가 많고, 그 중간 단계에서 만들어진 객체들이 모두 버려진다면 **성능 문제**가 더 불거진다

#### 성능 문제 해결방안
1. 다단계 연산들을 예측하여 기본 기능으로 제공한다
   - 다단계 연산들을 기본으로 제공하면 더이상 각 단계마다 새로운 인스턴스를 생성할 필요가 없다
2. 다단계 연산 속도를 높여주는 가변 동반 클래스를 사용할 수 있다
   - 가변 동반 클래스를 직접 사용하기란 어렵지만, 공개 클래스가 이 모든 것을 처리해준다
   - 클라이언트가 원하는 복잡한 모든 연산을 예측할 수 없다면 가변 동반 클래스를 공개할 수 있다
	   - String의 가변 동반 클래스는 StringBuilder이다.

### 불변 클래스를 만드는 방법
#### 1. 상속 막기
- 상속하지 못하게 막는 가장 단순한 방법은 final 클래스로 선언하는 것이다
- 더 유연한 방법은 모든 생성자를 prvate 혹은 package-private으로 선언하고, public 정적 팩터리 메서드를 구현하는 것이다
	- 다수의 구현 클래스를 활용한 유연성을 제공한다
	- 객체 캐싱 기능을 사용해 성능을 끌어올릴 수 있다

### 불변 클래스 생성에서의 타협
- 불변 클래스 규칙에 따르면 모든 필드가 final이고 어떤 메서드도 그 객체를 수정할 수 없어야 한다
- 이 규칙은 과한 감이 있어서 '어떤 메서드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없다'로 수정할 수 있다

### 정리
 - 클래스는 꼭 필요한 경우가 아니면 불변이어야 한다
	 - 게터가 있다고해서 무조건 세터를 만들지 말자
 - 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분은 최소한으로 줄이자
	 - 객체가 가질 수 있는 상태의 수가 줄어들면 오류가 생길 가능성도 줄어든다
	 - 다른 합당한 이유가 없다면 필드는 private final이어야 한다
 - 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다
	 - 확실한 이유가 없다면 생성자와 정적 패겉리 외에는 어떤 초기화 메서드도 public으로 제공하면 안된다



