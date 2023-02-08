# 디폴트 메서드
- 인터페이스를 구현하는 클래스는 인터페이스에서 정의하는 모든 메서드 구현을 제공하거나 아니면 슈퍼 클래스의 구현을 상속받아야 한다
- 라이브러리 설계자 입장에서 인터페이스에 새로운 메서드를 추가하는 등 인터페이스를 바꾸고 싶을 때 문제가 발생한다
  - 이전 인터페이스를 구현했던 모든 클래스의 구현을 변경해야 한다
- 자바 8에서 인터페이스의 이러한 단점을 해결하기 위해 기존 인터페이스를 구현하는 2가지 방식을 제공한다
  - 인터페이스 내부에 정적 메서드를 사용한다
  - 인터페이스의 기본 구현을 제공할 수 있도록 디폴트 메서드를 제공한다
- List 인터페이스의 sort는 디폴트 메서드이다

```java
default void sort(Comparator<? super E> c) {
    Collections.sort(this, c);
}
```

```java
List<Integer> numbers = Arrays.asList(3, 5, 1, 2, 6);
numbers.sort(Comparator.naturalOrder());
```

- Comparator.naturalOrder()는 표준 앏파벳 순서로 요소를 정렬할 수 있도록 Comparator 객체를 반환하는 Comparator 인터페이스에 추가된 새로운 정적메서드이다.
- 디폴트 메서드를 이용하면 인터페이스의 기본 구현을 그대로 상속하므로 인터페이스에 자유롭게 새로운 메서드를 추가할 수 있게된다.


## 변화하는 API
- API를 설계하면 인터페이스를 구현하는 클래스들이 생겨난다.
- 뒤 늦게 인터페이스에 부족한 기능을 발견하고 추가하는 경우가 생길 수 있다
- 인터페이스에 새로운 메서드를 추가하더라도 바이너리 호환성은 유지된다
  - 바이너리 호환성은 새로 추가된 메서드를 호출하지만 않으면 새로운 메서드 구현이 없이도 기존 클래스 파일 구현이 잘 동작한다는 의미이다
  - 하지만 언젠가는 누군가가 새로 추가된 메서드를 사용하도록 코드를 변경할 수도 있다
  - 이때 런타임 에러가 발생할 수 있다(
  - 그리고 새로운 메서드를 구현하지 않은 채로 재 빌드하면 컴파일 에러가 발생할 수 있다

### 바이너리 호환성, 소스 호환성, 동작 호환성
- 바이너리 호환성
  - 뭔가를 바꾼 이후에도 에러 없이 기존 바이너리가 실행될 수 있는 상황
  - 인터페이스에 메서드를 추가했을 때 추가된 메서드를 호출하지 않는 한 문제가 일어나지 않는다
- 소스 호환성
  - 코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일할 수 있음을 의미함
  - 인터페이스에 메서드를 추가하면 클래스를 고쳐야하므로 소스 호환성이 아니다
- 동작 호환성
  - 코드를 바꾼 다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행하는 것을 의미함
  - 인터페이스에 메서드를 추가하더라도 프로그램에서 추가된 메서드를 호출할 일은 없으므로 동작 호환성을 유지된다

## 디폴트 메서드란 무엇인가?
- 디폴트 메서드는 인터페이스 자신을 구현하는 클래스에서 메서드를 구현하지 않을 수 있는 새로은 메서드 시그니처를 제공한다
- 인터페이스가 구현을 가질 수 있고, 인터페이스를 구현하는 클래스가 디폴트 메서드와 같은 메서드 시그니처를 정의하거나 디폴트 메서드를 오버라이드하는 문제를 해결하기 위한 규칙이 몇가지 존재한다

### 추상 클래스 VS 인터페이스
- 둘 다 추상 메서드와 바디를 포함하는 메서드를 정의할 수 있다
- 클래스는 하나의 추상클래스만 상속받을 수 있지만, 인터페이스를 여러 개 구현할 수 있다
- 추상 클래스는 인스턴스 변수로 공통 상태를 가질 수 있다. 인터페이스는 인스턴스 변수를 가질 수 없다


## 디폴트 메서드 활용 패턴
- 선택형 메서드(optional method)
- 동작 다중 상속(multiple inheritance of behavior)

### 선택형 메서드
- 디폴트 메서드를 이용하면 메서드에 기본 구현을 제공할 수 있다
  - 자바 8 이전에는 Iterator의 remove를 사용하지 않아 많은 구현체들은 해당 메서드에 빈 구현을 제공했다

```java
interface Iterator<T> {
  boolean hasNext();
  T next();
  default void remove() {
    throw new UnsupprtedOperationException(); // 기본 구현 제공
  }
}
```

### 동작 다중 상속
- 디폴트 메서드를 사용하면 기존에는 불가능 헀던 동작 다중 상속 기능을 구현할 수 있다
  - 클래스는 단일 상속만 가능하지만, 인터페이스는 다중 구현이 가능하다

![image](https://user-images.githubusercontent.com/60502370/177255155-971b9b46-ffe6-4da0-a6de-3fb6f7449af3.png)

**다중 상속 형식**

- ArrayList는 한 개의 클래스를 상속 받고, 여섯 개의 인터페이스를 구현한다
  - ArrayList는 AbstractList, List, RandomAccess, Cloneable, Serializable, Iterable, Collection의 서브형식(Subtype)이 된다
  - 따라서 디폴트 메서드를 사용하지 않아도 다중 상속을 활용할 수 있다(?)
- 자바 8에서는 인터페이스가 구현을 포함할 수 있으므로 클래스는 여러 인터페이스에서 동작을 상속받을 수 있다
  - 중복되지 않은 최소한의 인터페이스를 유지한다면 동작을 쉽게 재사용하고 조합할 수 있다

**기능이 중복되지 않는 최소의 인터페이스**
- 다양한 특성을 갖는 여러 모양을 정의한다고 가정
  - 어떤 모양은 회전할 수 없지만, 크기는 조잘할 수 있다
  - 어떤 모양은 회전할 수 있으며 움직일 수 있지만 크기는 조절할 수 없다

```java
public interface Rotatable {
  void setRotationAngle(int angleInDegrees);
  int getRotationAngle();
  default void rotateBy(int angleInDegrees) { // 기본 구현
    setRotationAngle((getRotationAngle() + angleInDegrees) % 360);
  }
}
```

- 구현해야 할 다른 메서드에 따라 뼈대 알고리즘이 결정되는 템플릿 디자인 패턴과 비슷해 보인다
  - Retatable을 구현하는 모든 클래스는 setRotationAngle, getRotationAngle은 구현해야 하지만, rotateBy는 구현하지 않아도 된다

```java
public interface Moveable {
  int getX();
  int getY();
  void setX(int x);
  void setY(int y);

  default void moveHorizontally(int distance) {
    set(getX() + distance);
  }

  default void moveVertically(int distance) {
    setY(getY() + distance);
  }
}
```

```java
public interface Resizable {
  int getWidth();
  int getHeight();
  void setWidth(int width);
  void setHeight(int height);
  void setAbsoluteSize(int width, int height);

  default void setRelativeSize(int wFactor, int hFactor) {
    setAbsoluteSize(getWidth() / wFactor, getHeight() / hFactor);
  }
}
```

**인터페이스 조합**

- 이제 인터페이스를 조합하여 움직일 수 있거나, 회전할 수 있거나, 크기를 조절할 수 있는 클래스를 만들 수 있다
- 구현 클래스는 인터페이스에 구현된 다양한 메서드를 호출할 수 있게 된다
- 구현 클래스에서 디폴트 메서드를 재정의하지 않으면, 디폴트 메서드를 효율적으로 리팩토링하면 구현 클래스도 해당 메서드를 사용할 수 있다는 장점이 있다

**옳지 못한 상속**
- 상속으로 코드 재사용 문제를 모두 해결할 수 있는 것은 아니다
  - 한 개의 메서드를 재사용하기 위해 100개의 필드와 메서드가 정의되어 있는 클래스를 상속하는 것은 좋은 생각이 아니다
  - 이때는 멤버 변수를 이용해서 클래스에서 필요한 메서드를 직접 호출하는 것이 좋다(델리게이션)


## 해석 규칙
- 같은 시그니처를 갖는 디폴트 메서드를 상속 받는 상황이 발생할 수 있다

```java
public interface A {
  default void hello() {
    System.out.println("Hello from A");
  }
}

public interface B extends A {
  default void hello() {
    System.out.println("Hello from B");
  }
}

public interface C implements B, A {
  public static void main(String... args) {
    new C().hello(); // ??
  }
}
```
- 이러한 다이아몬드 문제를 해결하기 위해 자바 8은 규칙을 제공한다

### 알아야할 세 가지 해결 규칙
1. 클래스가 항상 이긴다. 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 가진다
2. 1번 규칙 이외의 상황에서는 서브인터페이스가 이긴다. 상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브인터페이스가 이긴다. B가 A를 상속받는다면 B가 A를 이긴다
3. 여전히 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다

### 충돌 그리고 명시적인 문제 해결

```java
public interface A {
  default void hello() {
    System.out.println("Hello from A");
  }
}

public interface B {
  default void hello() {
    System.out.println("Hello from B");
  }
}
```

- 인터페이스간 상속관계가 없으므로 2번 규칙을 적용할 수 없어 에러가 발생한다

**충돌 해결**
- 두 인터페이스를 동시에 구현하는 구현 클래스는 명시적으로 메서드를 오버라이드 해야 한다.


