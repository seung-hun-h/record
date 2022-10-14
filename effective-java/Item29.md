## ITEM29 이왕이면 제네릭 타입으로 만들라
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
      Object result = elements[--size];  
      elements[size] = null;  
      return result;  
   }  
  
   private void ensureCapacity() {  
      if (elements.length == size) {  
         elements = Arrays.copyOf(elements, 2 * size + 1);  
      }  
   }  
}
```

- Stack 클래스를 제네릭으로 변경한다
	- 제네릭으로 바꾼다 하더라도 현재 버전을 사용하는 클라이언트에는 아무런 해가 없다

### 방법1:  제네릭 배열로 형변환
```Java
public class Stack<E> {  
	private static final int DEFAULT_INITIAL_CAPACITY = 16;  
	  
	private E[] elements;  
	private int size = 0;  
	  
	@SuppressWarnings("unchecked")  
	public Stack() {  
	   elements = (E[])new Object[DEFAULT_INITIAL_CAPACITY];  
	}  
	  
	public void push(E object) {  
	   ensureCapacity();  
	   elements[size++] = object;  
	}  
	  
	public Object pop() {  
	   if (size == 0) {  
	      throw new EmptyStackException();  
	   }  
	   Object result = elements[--size];  
	   elements[size] = null;  
	   return result;  
	}  
	  
	private void ensureCapacity() {  
	   if (elements.length == size) {  
	      elements = Arrays.copyOf(elements, 2 * size + 1);  
	   }  
	}
}
```

- 일단 비검사 형변환이 프로그램 타입의 안전성을 해치지 않음을 확인해야 한다
	- 컴파일러는 알 수 없다
- 비검사 형변환이 안전함을 직접 증명했다면 `@SupressWarnings`애노테이션을 사용해 해당 경고를 숨긴다
- 위 코드에서는 생성자에서 제네릭 배열로 형변환했다

### 방법2: 반환 원소 형변환
```Java
public class Stack<E> {
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
	  
	public E pop() {  
	   if (size == 0) {  
	      throw new EmptyStackException();  
	   }  
	   @SuppressWarnings("unchecked") E result = (E)elements[--size];  
	   elements[size] = null;  
	   return result;  
	}  
	  
	private void ensureCapacity() {  
	   if (elements.length == size) {  
	      elements = Arrays.copyOf(elements, 2 * size + 1);  
	   }  
	}
}
```

- `pop()`에서 비검사 형변환을 수행하는 문에 `@SuppressWarnings`애노테이션을 사용했다

- 현업에서는 방법1을 선호한다
	- 방법1은 형변환을 배열 생성 시 단 한번만 하는 반면, 방법2는 배열에서 원소를 읽을 때마다 형변환을 수행한다
- 하지만 방법1은 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염을 일으킨다

---

- '배열보다는 리스트를 우선하라'는 아이템 28과 모순돼 보인다
- 하지만 제네릭 타입안에서 리스트를 사용하는 게 항상 가능하지도, 꼭 더 좋은 것도 아니다
	- 자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList 같은 제네릭 타입도 결국은 기본 타입인 배열을 사용해 구현해야 한다
	- HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다