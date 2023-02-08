## ITEM19 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

>
>상속용 클래스를 설계하는 것은 어렵다. 세부 구현 사항을 문서로 남기고 그것을 반드시 지켜야 한다. 효율적인 하위 클래스를 만들 수 있도록 protected 메서드를 제공해야 할 수도 있다. 그러니 클래스를 확장해야 할 명확한 이유가 떠오르지 않으면 상속을 금지하는 편이 낫다. 상속을 금지하려면 클래스를 final로 선언하거나 생성자 모두를 외부에서 접근할 수 없도록 만들면 된다.
>

### 상속용 클래스는 문서화하자
- 문서화하지 않은 외부 클래스는 언제 어떻게 변경될 지 모르기 때문에 상속하기 위험하다
- 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 **문서로 남겨야한다**
	- 재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야 한다
	- 재정의 가능 메서드가 다른 재정의 가능 메서드를 호출할 수도 있다
- `@impleSpec` 태그를 붙여주면 'Implementation Requirements'로 시작하는 절을 자바독 도구가 만들어준다
	- 명령줄 매개 변수로 `-tag "impleSpec:a:Implementation Requirements"`를 지정해주면 된다
- 상속용 클래스를 문서화하면 좋은 API 문서의 조건 '어떻게가 아니라 무엇을 하는지 설명해라' 를 어기게 된다
	- 다시말해 내부 구현을 문서화한다
	- 하지만 클래스의 상속을 안전하게 하기  위해서는 내부 구현 방식을 설명해야만 한다

### 내부 동작 과정 중에 끼어들 수있는 훅을 선별하여 protected로 선언하자
- 효율적인 하위 클래스를 어려움 없이 만들기 위해서는 내부 동작 과정 중에 끼어들 수 있는 훅을 잘 선별하여 **protected 메서드 형태로 공개**해야할 수도 있다
	- `java.util.AbstractList`는 clear 메서드의 최적화를 위해 `removeRange`를 protected로 공개했다
	- 일반 사용자는 removeRange를 사용하지 않는다

### 상속용 클래스의 검증
- 상속용 클래스를 시험하는 방법은 **직접 하위 클래스를 만들어보는 것**이 유일하다
	- 하위 클래스를 여러개 만들 때까지 전혀 사용하지 않는 protected 멤버는 private이었어야 할 가능성이 크다

### 상속용 클래스의 제약

#### 상속용 클래스의 생성자는 직간접적으로 재정의 가능 메서드를 호출해서는 안된다

```Java
public class Super {  
   public Super() {  
      overrideMe();  
   }  
  
   public void overrideMe() {  
  
   }  
}
```

```Java
public class Sub extends Super{  
   private final Instant instant;  
  
   public Sub() {  
      this.instant = Instant.now();  
   }  
  
   @Override  
   public void overrideMe() {  
      System.out.println("instant = " + instant);  
   }  
  
   public static void main(String[] args) {  
      Sub sub = new Sub();  
      sub.overrideMe();  
   }  
}
```

```
instant = null
instant = 2022-09-26T22:48:37.285572Z
```

- 위 코드에서 하위 클래스의 생성자가 상위 클래스의 생성자를 먼저 호출하여 인스턴트 필드를 출력한다
- final 필드는 일반적으로 반드시 단 하나의 상태를 가져야함에도 두 가지 상태를 가지게 되었다

#### Cloneable과 Serializable 인터페이스는 상속용 설계의 어려움을 더한다
- 둘 중 하나라도 구현한 클래스를 상속용으로 설계한다는 것은 일반적으로 좋지 않은 생각이다
- clone과 readObject는 위에서 본 생성자와 비슷한 효과를 낸다
	- clone과  readObject는 직간접적으로 재정의 가능한 메서드를 호출하면 안된다
	- readObject는 하위 클래스가 역직렬화 되기전에 재정의한 메서드부터 호출하게 된다
	- clone은 하위 클래스의 clone이 복제본의 상태를 수정하기 전에 재정의한 메서드를 호출하게 된다
	- 특히 clone이 잘못되면 복제본뿐 아니라 원본 객체에도 피해를 줄 수 있다

```Java
public class Super implements Cloneable {  
  
   Super() {  
   }  
  
   @Override  
   public Super clone() throws CloneNotSupportedException {  
      final Super clone = (Super) super.clone();  
      clone.doSomething();  
      return clone;  
   }  
   void doSomething() { // Overridable  
      System.out.println("doSomething from origin");  
   }  
}
```

```Java
public class Sub extends Super {  
   Sub() {  
   }  
  
   @Override  
   void doSomething() {  
      System.out.println("doSomething from clone");  
   }  
  
   public static void main(String[] args) throws CloneNotSupportedException {  
      Super bc = new Sub();  
      bc.clone();  
   }  
}
```

```
doSomething from clone
```

- Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 갖는다면 이 메서드들은 private이 아닌 protected로 선언해야 한다
	- private이면 하위 클래스에서 무시된다

#### 상속용으로 설계하지 않은 클래스는 상속을 금지하자
- 일반적인 구체 클래스는 final도 아니고, 상속용으로 설계되거나 문서화되지도 않았다
- 그대로 두면 클래스에 변화가 생길 떄마다 하위 클래스를 오동작하게 만들 수 있다
- 따라서 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이 좋다
	- 두 번째 선택지로는 생성자를 private으로 선언하고 정적 팩터리 메서드를 만들어줄 수 있다
		- 내부에서 다양한 하위 클래스를 만들어줄 수 있다
	- 세 번째 선택지로는 래퍼 클래스가 있다
		- 상속 대신 기능을 증강할 수 있다
- 다만 클래스 내부에서 재정의 가능한 메서드를 사용하지 않게하고 이를 문서화하면 상속을 허용할 수도 있다
- 클래스의 동작을 유지하면서 재정의 가능 메서드를 사용하는 코드를 제거할 수 있는 방법은 'private 도우미 메서드'를 사용하는 것이다
	- 상위 클래스의 메서드를 문제없이 호출할 수 있거나
	- 하위 클래스에서 완전히 재정의할 수 있다