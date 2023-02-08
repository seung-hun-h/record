## ITEM13 clone의 재정의는 주의해서 진행하라
>
>새로운 인터페이스를 만들때 Cloneable을 구현하거나 확장하지 마라. 대신 복사 생성자/팩터리를 사용하라. 단, 배열만은 clone 메서드 방식이 가장 깔끔한, 이 규칙에 합당한 예외라 할 수 있다
>

- `Cloneable`은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이지만, 아쉽게도 목적을 제대로 달성하지 못했다
	- `clone`이 선언된 곳이 Cloneable이 아닌 `Object`이다
	- Cloneable을 구현하는 것만으로 외부 객체에서 clone 메서드를 호출할 수 없다
	- 해당 객체가 **접근이 허용된 clone 메서드를 제공한다는 보장이 없기 때문**에 리플렉션을 사용한다고 해서 100% 성공한다고 말할 수 없다

### Cloneable 인터페이스
- Cloneable 인터페이스는 Object의 clone 메서드의 동작 방식을 결정한다
- clone을 호출하면 해당 클래스의 필드를 하나하나 복제한 객체를 반환하며, 그렇지 않으면 `CloneNotSupportedException`을 던진다
	- 이는 일반적인 방식의 구현은 아니다. Cloneable의 경우에는 상위 클래스에 정의된 protected 메서드의 동작 방식을 변경한 것이다
- 실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 제대로 복제가 이루어지리라 기대한다
	- 이를 위해서는 허술한 프로토콜을 지켜야 한다
	- 허술한 프로토콜로 인해 깨지기 쉽고, 위험하고, 모순적인 매커니즘이 탄생한다. 생성자를 호출하지 않고도 객체를 생성할 수 있게 된다

**clone 메서드의 일반 규약**
>이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은  참이다
>
>x.clone() != x
>x.clone().getClass() = x.getClass()
>
>하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다
>한편 다음 식도 일반적으로 참이지만 필수는 아니다
>
>x.clone().equals(x)
>
>관례상 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다. 이 클래스와 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다
>
>x.clone().getClass() = x.getClass()
>
>관례상 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다
>

- 강제성이 없다는 점만 빼면 생성자 연쇄와 살짝 비슷한 매커니즘이다
	- clone 메서드가 super.clone이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 불평하지 않을 것이다. 하지만 이 클래스의 하위 클래스에서 super.clone을 호출한다면 잘못된 클래스의 객체가 만들어져, 결국 하위 클래스의 clone 메서드가 제대로 동작하지 않게 된다
		- super.clone을 연쇄적으로 호출하도록 하면 최종적으로 최상위 클래스 타입을 반환할 수 밖에 없다

### 제대로 동작하는 clone
- 먼저 super.clone을 호출한다
	- 여기서 반환된 객체는 상위 객체을 완벽하게 복사한 객체일 것이다
- 자바가 공변 반환 타이핑을 지원하는 점을 활용하여 하위 클래스로 타입 변환하고 반환한다
	- 공변 반환 타이핑은 jdk 1.5부터 지원하는 개념으로 부모 클래스의 반환 타입을 자식 클래스의 반환 타입으로 변경할 수 있게 하는 것이다

```Java
@Override
public PhoneNumber clone() {
	try {
		return (PhoneNumber) super.clone();
	} catch (CloneNotSupportedException e) {
		throw new AssertionError();
	}
}
```

### clone은 불변식을 보장해야 한다
- 클래스가 가변 객체를 참조할 때 clone으로 인해 문제가 발생할 수 있다

```Java
public class Stack {  
   private static final int DEFAULT_INITIAL_CAPACITY = 16;  
  
   private Object[] elements;  
   private int size = 0;  
  
   public Stack() {  
      elements = new Object[DEFAULT_INITIAL_CAPACITY];  
   }  
  
   public void push(Object object) {  
      ensureCapacity();  
      elements[size++] = object;  
   }  
  
   public Object pop() {  
      if (size == 0) {  
         throw new EmptyStackException();  
      }  
      Object result = elements[size];  
      elements[size--] = null;  
      return result;  
   }  
  
   private void ensureCapacity() {  
      if (elements.length == size) {  
         elements = Arrays.copyOf(elements, 2 * size + 1);  
      }  
   }
}
```

- Stack에서 size는 제대로 복사되었을 지라도, 복사된 객체는 동일한 elements를 참조하고 있을 것이다
	- 따라서 원본이나 복제에서 데이터를 변경하면 서로 영향을 미쳐 의도한 대로 동작하지 않는다
- clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, **clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다**

```Java
@Override  
public Stack clone() {  
   try {  
      Stack result = (Stack)super.clone();  
      result.elements = elements.clone();  
      return result;  
   } catch (CloneNotSupportedException e) {  
      throw new AssertionError();  
   }  
}
```

- 만약 elements가 final로 선언되어 있었다면 위 방식은 동작하지 않는다
- Cloneable 아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다
	- 복제할 수 있는 클래스를 만들기위해 일부 필드에서 final을 제거해야할 수 있다

### clone의 반복적인 재귀 호출로 문제가 발생할 수 있다
- clone을 재귀호출 하는 것만으로 부족한 경우가 있다
- 연결리스트를 복제하는 경우가 그 예가 될 수 있는데, 자바 컬렉션에서 제공하는 연결리스트가 아니라 직접 구현하며 알아본다

```Java
public class HashTable implements Cloneable {  
   private static final int SIZE = 20;  
   private Entry[] buckets = new Entry[SIZE];  
   private static class Entry {  
      final Object key;  
      Object value;  
      Entry next;  
  
      public Entry(Object key, Object value, Entry next) {  
         this.key = key;  
         this.value = value;  
         this.next = next;  
      }  
   }  
  
   @Override  
   public HashTable clone() {  
      try {  
         HashTable result = (HashTable)super.clone();  
         result.buckets = buckets.clone();  
         return result;  
      } catch (CloneNotSupportedException exception) {  
         throw new AssertionError();  
      }  
   }  
	// 생략
}
```

- 위 코드에서 복제본은 자신만의 버킷 배열을 가지지만, 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 수 있다
- 이를 해결하기 위해서는 각 버킷을 구성하는 연결 리스트를 복사 해야한다

```Java
public class HashTable implements Cloneable {  
   private static final int SIZE = 20;  
   private Entry[] buckets = new Entry[SIZE];  
  
   private static class Entry {  
      final Object key;  
      Object value;  
      Entry next;  
  
      public Entry(Object key, Object value, Entry next) {  
         this.key = key;  
         this.value = value;  
         this.next = next;  
      }  
        
      Entry deepCopy() {  
         return new Entry(key, value, next == null ? null : next.deepCopy());  
      }   
   }  
  
   @Override  
   public HashTable clone() {  
      try {  
         HashTable result = (HashTable)super.clone();  
         result.buckets = new Entry[SIZE];  
  
         for (int i = 0; i < SIZE; i++) {  
            if (buckets[i] != null) {  
               result.buckets[i] = buckets[i].deepCopy();  
            }  
         }  
           
         return result;  
      } catch (CloneNotSupportedException exception) {  
         throw new AssertionError();  
      }  
   }  
}
```

- HashTable.Entry가 깊은 복사(deepCopy)를 하도록 보강되었다
- Entry는 깊은 복사를 위해서 재귀 호출하고 있는데, 이 때문에 리스트의 원소 수만큼 스택 프레임을 소비하여 리스트가 길면 스택 오버플로를 발생시킬 위험이 있다
- 이 문제를 피하려면 deepCopy를 재귀 호출대신 반복자를 써서 순회하는 방식으로 수정해야 한다

```Java
Entry deepCopy() {  
   Entry result = new Entry(key, value, next);  
   for (Entry p = result; p.next != null; p = p.next) {  
      p.next = new Entry(p.next.key, p.next.value, p.next.next);  
   }  
   return result;  
}
```

### 고수준 API를 호출하여 clone 구현하기
- clone을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한 다음, 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출한다
- HashTable 예에서라면, buckets 필드를 새로운 버킷 배열로 초기화한 다음 원본 테이블에 담긴 모든 키-값 쌍 각각에 대해 복제본 테이블의 put(key, value)를 호출해 둘의 내용을 똑같이 하면된다
-  이 방식은 고수준 API를 호출하기 때문에 비교적 속도가 느리고, Cloneable 아키텍처의 기초가 되는 필드 단위 객체 복사를 위회한다

### clone을 구현할 때 더 알아두어야 할 것들
- 생성자와 마찬가지로 clone에서는 재정의 될 수 있는 메서드를 호출하지 말아야 한다
	- final이나 private이어야 한다
- public인 clone 메서드에서는 throws 절을 없애야 한다
- 상속해서 쓰기 위한 클래스에서는 Cloneable을 구현하지 말아야 한다
- 다른 방법으로는 clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의 하지 못하게 할 수 있다

```Java
@Override
protected final Object clone() throws CloneNotSupportedException {
	throw new CloneNotSupportedException();
}
```

- Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드 역시 적절히 동기화 해주어야 한다


### clone 구현 요약
- Cloneable을 구현하는 모든 클래스는 clone을 재정의 해야 한다
- 접근 제한자는 public, 반환 타입은 본인으로 변경한다
- super.clone을 가장 먼저 호출한 다음 필요한 필드를 전부 적절히 수정한다
	- 객체의 내부 깊은 구조에 숨어있는 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체를 가리키게 한다
- 내부 복사는 clone을 재귀적으로 호출하는 것이 일반적이지만 항상 그런것은 아니다

### 복사 생성자와 복사 팩터리
- 항상 Cloneable을 구현하여 복사 객체를 반환할 필요는 없다
- 복사 생성자와 복사 팩터리라는 더 나은 방법이 있다

```Java
public Yum(Yum yum) {
...
}

public static Yum newInstance(Yum yum) {
...
}
```

- 복사 생성자/팩터리는 Cloneable을 구현하는 방식보다 나은 면이 많다
	- 언어 모순적이고 위험천만한 객체 생성 매커니즘을 사용하지 않는다
	- 엉성하게 문서화된 규약에 기대지 않는다
	- 정상적인 final 필드 용법과도 충돌하지 않는다
	- 불필요한 검사 예외를 던지지 않는다
	- 형변환도 필요치 않다
	- 복사 생성자/팩터리는 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다
		- 관례상 모든 범용 컬렉션 구현체는 Collection이나 Map 타입을 받는 생성자를 제공한다
		- 예를 들어, HashSet 객체 s를 TreeSet 타입으로 복제할 수 있다
