## 과도한 동기화는 피하라
---
- 과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠트리고, 예측할 수 없는 동작을 낳기도한다
- 응답 불가와 안전 실패를 피하려면 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안된다

### 예제
```Java
public class ObservableSet<E> extends ForwardingSet<E> {  
   public ObservableSet(Set<E> s) {  
      super(s);  
   }  
  
   private final List<SetObserver<E>> observers = new ArrayList<>();  
  
   public void addObserver(SetObserver<E> observer) {  
      synchronized (observers) {  
         observers.add(observer);  
      }  
   }  
  
   public void removeObserver(SetObserver<E> observer) {  
      synchronized (observers) {  
         observers.remove(observer);  
      }  
   }  
  
   private void notifyElementAdded(E element) {  
      synchronized (observers) {  
         for (SetObserver<E> observer : observers) {  
            observer.added(this, element);  
         }  
      }  
   }  
  
   @Override  
   public boolean add(E e) {  
      boolean added = super.add(e);  
      if (added) {  
         notifyElementAdded(e);  
      }  
      return added;  
   }  
  
   @Override  
   public boolean addAll(Collection<? extends E> c) {  
      boolean result = false;  
      for (E e : c) {  
         result |= add(e);  
      }  
      return result;  
   }  
}
```

```Java
@FunctionalInterface  
public interface SetObserver<E> {  
   void added(ObservableSet<E> set, E element);  
}
```

```Java
ObservableSet<Object> observableSet = new ObservableSet<>(new HashSet<>());  
  
observableSet.addObserver((s, e) -> System.out.println("e = " + e));  
  
for (int i = 0; i < 100; i++) {  
   observableSet.add(i);  
}
```

- 해당 코드를 실행하면 집합에  추가된 원소가 잘 출력될 것이다

### ConcurrentModificationException 발생 예제
```Java
ObservableSet<Integer> observableSet = new ObservableSet<>(new HashSet<>());  
  
observableSet.addObserver(new SetObserver<>() {  
   @Override  
   public void added(ObservableSet<Integer> set, Integer element) {  
      System.out.println("element = " + element);  
  
      if (element == 23) {  
         set.removeObserver(this); // ConcurrentModificationException  
      }  
   }  
});  
  
for (int i = 0; i < 100; i++) {  
   observableSet.add(i);  
}
```

- 위 예제를 실행하면 `ConcurrentModificationException`이 발생한다
	- 컬렉션을 조회하는 중에 컬렉션이 변경되면 발생하는 예외이다
- `notifyElementAdded` 메서드에서 컬렉션을 조회하는 중에 `removeObserver`메서드가 호출되어 예외가 발생한다

### 교착상태 예제
```Java
ObservableSet<Integer> observableSet = new ObservableSet<>(new HashSet<>());  
  
observableSet.addObserver(new SetObserver<>() {  
   @Override  
   public void added(ObservableSet<Integer> set, Integer element) {  
      System.out.println("element = " + element);  
  
      if (element == 23) {  
         ExecutorService executorService = Executors.newSingleThreadExecutor();  
         try {  
            executorService.submit(() -> set.removeObserver(this)).get();  
         } catch (ExecutionException | InterruptedException exception) {  
            throw new AssertionError(exception);  
         } finally {  
            executorService.shutdown();  
         }  
      }  
   }  
});  
  
for (int i = 0; i < 100; i++) {  
   observableSet.add(i);  
}
```

- 백그라운드 스레드가 `set.removeObserver()` 호출 시 락을 가지려 하지만, 메인 스레드가 락을 가지고 있고, 동시에 메인 스레드는 백그라운드 스레드가 `removeObserver` 메서드를 완료하기를 기다린다

### synchronized의 reentrant
- 자바의 `synchronized`는 재진입이 가능하다
- 위 예제에서 `notifyElementAdded`에서 락을 가졌기 때문에 `removeObserver`도 호출이 가능하다
- 재진입 가능 락은 객체 지향 멀티스레드 프로그램을 쉽게 구현할 수 있도록 해주지만, 응답 불가가 될 상황을 안전 실패로 변모 시킬 수 있다.

### 재진입 가능 락의 문제점 해결
- 외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 된다
```Java
private void notifyElementAdded(E element) {  
   List<SetObserver<E>> snapShot = null;  
   synchronized (observers) {  
      snapShot = new ArrayList<>(observers);  
   }  
  
   for (SetObserver<E> observer : snapShot) {  
      observer.added(this, element);  
   }  
}
```

- `snapShot`을 순회하므로 `ConcurrentModificationException` 이 발생하지 않는다
- 복사본을 생성할 때만 락을 가지기 때문에 `set.removeObserver`가 완료될 때까지 기다리지 않아 교착상태가 발생하지 않는다
- 동기화 영역 바깥에서 호출되는 외계인 메서드를 열린 호출이라 한다
	- 외계인 메서드는 언제 실행이 끝날지 알 수 없는데, 동기화 영역안에서 호출된다면 다른 스레드는 대기해야 한다
	- 따라서 열린 호출은 실패 방지 효과뿐 아니라 동시성 효율을 크게 높여준다

- 위 처럼 로직을 작성할 수도 있지만 `CopyOnWriteArrayList`를 사용할 수도 있다
	- 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현했다
	- 내부의 배열을 수정되지 않으니 순회할 때 락이 필요 없어 매우 빠르다
	- 수정할 일이 드물고 순회만 빈번이 일어나는 관찰자 리스트 용도로는 최적이다
```Java
public class ObservableSet<E> extends ForwardingSet<E> {  
   public ObservableSet(Set<E> s) {  
      super(s);  
   }  
  
   private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();  
  
   public void addObserver(SetObserver<E> observer) {  
      observers.add(observer);
   }  
  
   public boolean removeObserver(SetObserver<E> observer) {  
      return observers.remove();
   }  
  
   private void notifyElementAdded(E element) {  
	 for (SetObserver<E> observer : observers) {  
		observer.added(this, element);  
	 }  
   }  
  
   @Override  
   public boolean add(E e) {  
      boolean added = super.add(e);  
      if (added) {  
         notifyElementAdded(e);  
      }  
      return added;  
   }  
  
   @Override  
   public boolean addAll(Collection<? extends E> c) {  
      boolean result = false;  
      for (E e : c) {  
         result |= add(e);  
      }  
      return result;  
   }  
}
```

### 동기화의 기본 규칙
- 동기화의 기본 규칙은 동기화 영역에서는 가능한 한 일을 적게 하는 것이다
	- 동기화를 과도하게 하면 가상머신의 코드 최적화를 제한한다

### 가변 클래스를 작성할 때
- 다음 두 가지 선택 중 하나를 따른다
1. 동기화를 전혀 하지말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화 하게 만들자
   - `java.util`
2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자
   - `java.util.concurrent`
   - 클래스를 내부에서 동기화하기로 했다면, 락 분할, 락 스트라이핑, 비차단 동시성 제어 등 다양한 기법을 통해 동시성을 높여줄 수 있다
3. 여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기 전에 반드시 동기화 해야 한다