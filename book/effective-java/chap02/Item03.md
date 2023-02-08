## ITEM03 private 생성자나 열거 타입으로 싱글턴임을 보증하라
- 싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다
- 예로는 함수 같은 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트를 들 수 있다

## 싱글턴을 만드는 방법

### 1.  public static 필드 멤버
```Java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() {}
	...
}
```

- private 생성자는 INSTANCE를 초기화할 때 딱 한번 호출된다
- 다른 생성자가 없어 Elvis의 인스턴스를 새로 생성할 수 없으므로 Elvis의 인스턴스는 전체 시스템에서 단 하나뿐임이 보장된다
	- 단, 리플렉션을 사용하면 private 생성자에 접근하여 인스턴스를 생성할 수 있다
	- 이러한 문제를 해결하려면 생성자를 수정해 두 번째 인스턴스가 생성될 경우 예외를 던질 수 있다
- 장점
	- staic final이라 절대로 다른 객체를 참조할 수 없어 API에 명백히 드러난다
	- 간결하다

### 2. public static 정적  팩터리 메서드
```Java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() {}

	public static Elvis getInstance() {
		return INSTANCE;
	}
	
	...
	
}
```

- 장점
	- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다
		- 나중에 스레드 별로 다른 인스턴스를 반환하도록 수정할 수 있다
	- 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다
	- 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있다

**1, 2번 방법의 단점: 직렬화**
- 두 방법 모두 클래스의 인스턴스를 전체 시스템에서 하나임을 보장한다
- 하지만 직렬화후 역직렬화를 하면 이야기가 달라진다

```Java
public class Elvis implements Serializable {  
   private static final Elvis INSTANCE = new Elvis();  
  
   private Elvis() {  
  
   }  
  
   public static Elvis getInstance() {  
      return INSTANCE;  
   }  
}

```

```Java
public static void main(String[] args) {  
   Serializer serializer = new Serializer();  
   Elvis origin = Elvis.getInstance();  
  
   byte[] serialize = serializer.serialize(origin);  
   Elvis deserialized = (Elvis)serializer.deserialize(serialize);  
  
   System.out.println(origin == deserialized);  // false
}
```

- 위 처럼 원본 객체를 직렬화 한 후 다시 역직렬화 하면 두 인스턴스는 다르다
- 이를 해결하기 위해 직렬화 대상 클래스에  `readResolve` 메서드를 추가하면 된다

```Java
public class Elvis implements Serializable {  
   private static final Elvis INSTANCE = new Elvis();  
  
   private Elvis() {  
  
   }  
  
   public static Elvis getInstance() {  
      return INSTANCE;  
   }  
  
   @Serial  
   private Object readResolve() {  
      return INSTANCE;  
   }  
}
```

```Java
public static void main(String[] args) {  
   Serializer serializer = new Serializer();  
   Elvis origin = Elvis.getInstance();  
  
   byte[] serialize = serializer.serialize(origin);  
   Elvis deserialized = (Elvis)serializer.deserialize(serialize);  
  
   System.out.println(origin == deserialized);  //true
}
```

### 3. 열거 타입 

```Java
public enum Elvis {
	INSTANCE;
}
```
- 장점
	- 간결하다
	- 추가적인 노력 없이 직렬화 할 수 있다
	- 복잡한 직렬화 상황이나 리플렉션 공격에도 인스턴스가 하나임을 안정적으로 보장한다
- 대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만들 수 있는 가장 좋은 방법이다
- 만드려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다


