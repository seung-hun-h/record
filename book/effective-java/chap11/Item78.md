## 공유 중인 가변 데이터는 동기화해 사용하라
---
- 자바는 동기화를 위해 `synchronized` 키워드를 제공한다
	- 해당 메서드나 블록을 한 번에 한 스레드씩 수행하도록 한다

### 동기화의 기능
- 동기화의 주요 기능에는 두 가지가 있다

1. 배타적 실행
   - 한 스레드가 어떤 데이터를 변경하고 있을 때, 다른 스레드에서 해당 데이터를 참조할 수 없게한다
   - 동기화를 제대로 사용하면 어떤 메서드도 이 객체의 상태가 일관되지 않은 순간을 볼 수 없다
   
2. 한 스레드에서 만든 변화를 다른 스레드에서 확인
   - 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다
   - 한 스레드가 변경한 데이터를 다른 스레드가 바로 확인하도록 한다

- 자바 언어 명세에는 스레드가 필드를 읽을 때 항상 '수정이 완전히 반영된 값'을 얻는다고 보장한다
- 하지만 한 스레드가 저장한 값이 다른 스레드에게 '보이는가'는 보장하지 않는다

- **동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다**

### 동기화 예제
```Java
public class StopThread {  
   private static boolean stopRequested;  
   public static void main(String[] args) throws InterruptedException {  
      Thread backgroundThread = new Thread(() -> {  
         int i = 0;  
         while (!stopRequested) {  
            i++;`  
         }  
      });  
      backgroundThread.start();  
  
      Thread.sleep(1000);  
      stopRequested = true;  
   }  
}
```

- `stopRequested`는 공유 가변 데이터이다
	- 동기화 되지 않아 `stopRequested`가 true로 변경되어도 백그라운드 스레드가 종료되지 않는다

```Java
public class SynchronizedStopThread {  
   private static boolean stopRequested;  
  
   private static synchronized void requestStop() {  
      stopRequested = true;  
   }  
  
   private static synchronized boolean stopRequested() {  
      return stopRequested;  
   }  
   public static void main(String[] args) throws InterruptedException {  
      Thread backgroundThread = new Thread(() -> {  
         int i = 0;  
         while (!stopRequested()) {  
            i++;  
         }  
      });  
      backgroundThread.start();  
  
      Thread.sleep(1000);  
      requestStop();  
   }  
}
```

- 쓰기 메서드(`requestStop`)와 읽기 메서드(`stopRequested`) 모두 동기화했다
- 쓰기와 읽기 동작이 모두 동기화되지 않으면 동작을 보장하지 않는다

### volatile
```Java
public class StopThread {  
   private static volatile boolean stopRequested;  
   public static void main(String[] args) throws InterruptedException {  
      Thread backgroundThread = new Thread(() -> {  
         int i = 0;  
         while (!stopRequested) {  
            i++;`  
         }  
      });  
      backgroundThread.start();  
  
      Thread.sleep(1000);  
      stopRequested = true;  
   }  
}
```
- `stopRequested` 필드를 volatile으로 선언하면 동기화를 생략해도 된다
- `volatile`은 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다

### ++ 연산자와 AtomicXXX
```Java
private static volatile int sequence = 0;

public static int generatedSerialNumber() {
	return sequence++;
}
```

- volatile 한정자를 사용했지만 위 코드는 제대로 동작하지 않는다
- `++` 연산자가 `sequence` 필드에 두 번 접근하기 때문이다
	- 읽기, 쓰기
- `java.util.concurrent.atomic` 패키지의 `AtomicLong` 을 사용하면 해결할 수 있다
	- 락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨있다
```Java
private static volatile AtomicLong sequence = 0;

public static int generatedSerialNumber() {
	return sequence.getAndIncrement();
}
```

### 문제를 피하는 방법
- 제일 좋은 방법은 가변 데이터는 단일 스레드에서만 사용하는 것이다
- 한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때 해당 객체에 공유하는 부분만 동기화 해도된다
	- 이런 객체를 사실상 불변이라 한다
	- 정적 필드, volatile 필드, final 필드, 락을 통한 접근, 동시성 컬렉션을 사용할 수 있다
