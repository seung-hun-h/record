## ITEM18 상속보다는 컴포지션을 사용하라

>
>상속은 강력하지만 캡슐화를 해친다. 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야 한다. is-a 관계일 때도 안심할 수는 없다. 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 위해 설계되지 않았다면 여전히 문제가 될 수 있다. 상속의 취약점을 피하려면 컴포지션을 사용하자. 특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 그렇다. 래퍼 클래스는 하위 클래스보다 강력하다.
>

### 상속은 위험하다
- 상속은 코드를 재사용하는 강력한 수단이지만 항상 최선은 아니다
- 상위 클래스가 구현을 목적으로 설계되었고, 같은 프로그래머가 통제하는 패키지에 하위 클래스가 존재하는경우에는 상속이 적절할 수 있다
- 하지만 다른 패키지의 구체 클래스를 상속하는 일은 위험하다

### 상속은 캡슐화를 깨뜨린다
#### 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다
- `HashSet`가 생성된 후 더해진 원소의 개수를 알아내기 위해 `InstrumentHashSet`이라는 하위 클래스를 구현한다

```Java
public class InstrumentHashSet<E> extends HashSet<E> {  
   private int addCount = 0;  
  
   public InstrumentHashSet() {  
   }  
  
   public InstrumentHashSet(int initialCapacity, float loadFactor) {  
      super(initialCapacity, loadFactor);  
   }  
  
   @Override  
   public boolean add(E e) {  
      addCount++;  
      return super.add(e);  
   }  
  
   @Override  
   public boolean addAll(Collection<? extends E> c) {  
      addCount += c.size();  
      return super.addAll(c);  
   }  
  
   public int getAddCount() {  
      return addCount;  
   }  
}
```

```Java
public static void main(String[] args) {  
   InstrumentHashSet<Integer> ints = new InstrumentHashSet<>();  
   ints.addAll(List.of(1, 2, 3, 4, 5));  
  
   System.out.println(ints.getAddCount());  // 10
}
```

- `InstrumentHashSet`은 잘 구현된 것 처럼 보이지만, 실제로 원소를 추가한 후 `getAddCount`를 호출하면 엉뚱한 값을 반환한다
- 이러한 문제는 HashSet의 `addAll`이 내부적으로 `add` 를 호출하기 때문에 발생한 일이다
- 이처럼 자신의 다른 부분을 사용하는 '자기 사용(self-use)' 문제는 구체 구현에 해당하는 것이기 때문에 자바 플랫폼의 전반적인 정책인지, 다음 릴리즈에 수정될 지도 알 수 없다

- 대안으로 addAll의 구현방식을 변경할 수도 있다
	- 전달받은 컬렉션을 순회하며 add를 호출할 수 있다
	- 하지만 상위 클래스의 메서드 동작을 다시 구현하는 일은 어렵고, 시간도 더 들고, 오류를 내거나 성능이 떨어질 수도 있다

#### 상위 클래스에 새로운 메서드가 추가될 수 있다
- 보안때문에 컬렉션에 추가된 모든 원소가 특정 조건을 만족해야 하는 프로그램을 생각해본다
- 상위 클래스의 모든 메서드를 재정의해 조건 검사를  하면 될 것 같다
- 하지만 상위 클래스에 메서드가 추가되고 하위 클래스가 이를 재정의 하지 않았다면, 새로 추가된 메서드를 호출하는 순간 조건에 맞지 않는 원소가 추가될 수 있다

#### 하위 클래스에 새로운 메서드를 추가 한다
- 앞서 살펴 본 두 문제 모두 메서드 재정의로 인해 발생했다
- 이를 해결하기 위해 클래스를 확장 하기 위해 메서드를 재정의 하지 않고 하위 클래스에 새로운 메서드를 추가할 수 있다
- 이러한 방법이 훨씬 안전한 것은 맞지만, 우연히 상위 클래스에 추가된 새로운 메서드가 하위 클래스의 메서드 와 시그니처가 같고 반환 타입이 다른 경우 프로그램은 컴파일 조차 되지 않는다
	- 반환 타입이 같은 경우 다시 재정의로 인한 문제가 발생할 수 있다

### 컴포지션을 사용하자
- 상속으로 인해 발생할 수 있는 여러 문제를 해결할 수 있는 방법이 컴포지션을 사용하는 것이다
	- 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다
	- 새 클래스의 인스턴스 메서드들은 기존 클래스에 대응하는 메서드를 호출한다
	- 이 방식을 전달(forwarding)이라 하며, 새 클래스의 메서드들을 전달 메서드(forwarding method)라 한다
- 이러한 방식은 기존 클래스의 내부 구현 방식에서 벗어나며, 새로운 클래스에 메서드를 추가하더라도 문제가 발생하지 않는다

```Java
public class ForwardingSet<E> implements Set<E> {  
   private final Set<E> s;  
  
   public ForwardingSet(Set<E> s) {  
      this.s = s;  
   }  
  
   @Override  
   public int size() {  
      return s.size();  
   }  
  
   @Override  
   public boolean isEmpty() {  
      return s.isEmpty();  
   }  
  
   @Override  
   public boolean contains(Object o) {  
      return s.contains(o);  
   }  
  
   @Override  
   public Iterator<E> iterator() {  
      return s.iterator();  
   }  
  
   @Override  
   public Object[] toArray() {  
      return s.toArray();  
   }  
  
   @Override  
   public <T> T[] toArray(T[] a) {  
      return s.toArray(a);  
   }  
  
   @Override  
   public boolean add(E e) {  
      return s.add(e);  
   }  
  
   @Override  
   public boolean remove(Object o) {  
      return s.remove(o);  
   }  
  
   @Override  
   public boolean containsAll(Collection<?> c) {  
      return s.containsAll(c);  
   }  
  
   @Override  
   public boolean addAll(Collection<? extends E> c) {  
      return s.addAll(c);  
   }  
  
   @Override  
   public boolean retainAll(Collection<?> c) {  
      return s.retainAll(c);  
   }  
  
   @Override  
   public boolean removeAll(Collection<?> c) {  
      return s.removeAll(c);  
   }  
  
   @Override  
   public void clear() {  
      s.clear();  
   }  
}
```

```Java
public class InstrumentSet<E> extends ForwardingSet<E> {  
   private int addCount = 0;  
  
   public InstrumentSet(Set<E> s) {  
      super(s);  
   }  
  
   @Override  
   public boolean add(E e) {  
      addCount++;  
      return super.add(e);  
   }  
  
   @Override  
   public boolean addAll(Collection<? extends E> c) {  
      addCount += c.size();  
      return super.addAll(c);  
   }  
  
   public int getAddCount() {  
      return addCount;  
   }  
}
```

```Java
public static void main(String[] args) {  
   InstrumentSet<Integer> ints = new InstrumentSet<>(new HashSet<>());  
   ints.addAll(List.of(1, 2, 3, 4, 5));  
  
   System.out.println(ints.getAddCount());  // 5
}
```

- 다른 Set을 감싸고 있다는 뜻에서 InstrumentSet 같은 클래스를 래퍼 클래스라 부르며, 기능을 덧씌운다는 의미에서 데코레이터 패턴이라 한다
- 컴포지션과 전달의 조합은 넓은 의미로 위임이라 한다
	- 엄밀히 따지면 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우에만 위임에 해당한다(?)

#### 래퍼 클래스의 단점
- 래퍼 클래스가 콜백 프레임워크와는 어울리지 않는다는 점을 주의해야 한다
- 콜백 프레임워크는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출 때 사용하도록 한다
- 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다
	- 이를 SELF 문제라고 한다

### 상속을 사용할 때 주의할 점
- 상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에만 쓰여야 한다
	- 클래스 B가 클래스 A와 'is-a' 관계일 때만 사용해야 한다
- 컴포지션을 사용해야할 때 상속을 사용하면 내부 구현을 불필요하게 노출하는 꼴이다
	- 클라이언트가 노출된 내부에 직접 접근할 수 있다
	- 사용자에게 혼란을 준다
		- Properties는 HashTable을 상속했는데, `getProperty(key)`와 `get(key)` 중 뭘 사용해야 할 지 헷갈린다
	- 상위 클래스를 직접 사용하면 하위 클래스의 불변식을 해칠 수 있다
		- Properties는 문자열만 허용하도록 설계하려 했으나, HashTable의 메서드를 호출하면 불변식이 깨진다
- 상속을 사용하기로 결정하기 전 스스로 자문 해라
	- 확장 하려는 클래스의 API에 아무런 문제가 없는가
	- 결함이 있다면, 이 결함이 여러분 클래스 API까지 전파되어도 괜찮은가
	- 컴포지션은 결함을 숨길 수 있지만, 상속은 결함까지 승계한다

