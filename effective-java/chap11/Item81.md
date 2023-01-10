## wait와 notify보다는 동시성 유틸리티를 사용하라
---
- wait와 notify는 올바르게 사용하기 아주 까다롭기에 고수준 동시성 유틸리티를 사용하자
- `java.util.concurrent`의 고수준 유틸리티는 세 범주로 나눌 수 있다
	- 실행자 프레임 워크
	- 동시성 컬렉션
	- 동기화 장치

### 동시성 컬렉션
- List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다
- 동시성 컬렉션에서 동시성을 무력화하는 것은 불가능하고 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다

#### ConcurrentHashMap
- `ConcurrentHashMap`은 동시성이 뛰어나며 속도도 무척 빠르다
- 동시성 컬렉션은 동기화한 컬렉션을 낡은 유산으로 만들었다
	- `Collections.synchronizedMap` 보다 `ConcurrentHashMap`을 사용하는 것이 훨씬 좋다

#### BlockingQueue
- `take`는 큐의 첫 원소를 꺼낸다. 만약 큐가 비었다면 새로운 원소가 추가될 때까지 기다린다


### 동기화 장치
- 자주 사용하는 동기화 장치
	- CountDownLatch
	- Semaphore
- 조금 덜 쓰이는 동기화 장치
	- CyclicBarrier
	- Exchanger

#### CountDownLatch
- 생성자에 int 값을 받으며, 이 값이 래치의 countDown 메서드를 몇번 호출해야 대기 중인 스레드를 깨우는지 결정한다

```Java
public class CountDown {  
   public static void main(String[] args) throws InterruptedException {  
      Executor executor = Executors.newFixedThreadPool(10);  
      long time = time(executor, 10, () -> System.out.println(Thread.currentThread().getId()));  
      System.out.println("time = " + time);  
   }  
  
   private static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {  
      CountDownLatch ready = new CountDownLatch(concurrency);  
      CountDownLatch start = new CountDownLatch(1);  
      CountDownLatch done = new CountDownLatch(concurrency);  
  
      for (int i = 0; i < concurrency; i++) {  
         executor.execute(() -> {  
            // 타이머에게 준비를 마쳤음을 알린다  
            ready.countDown();  
            try {  
               // 모든 작업자 스레드가 준비될 때까지 기다린다  
               start.await();  
               // 작업을 시작한다  
               action.run();  
            } catch (InterruptedException e) {  
               Thread.currentThread().interrupt();  
            } finally {  
               // 타이머에게 작업을 마쳤음을 알린다  
               done.countDown();  
            }  
         });  
      }  
  
      // 모든 작업자가 준비될 때까지 기다린다  
      ready.await();  
      long startNanos = System.nanoTime();  
      // 작업자들을 꺠운다  
      start.countDown();  
      // 모든 작업자가 일을 마치기까지 기다린다  
      done.await();  
      return System.nanoTime() - startNanos;  
   }  
}
```

- `System.currentTimeMillis`가 아닌 `System.nanoTime`을 사용하는 것이 좋다
	- 더 정확하고 정밀하며 시스템의 실시간 시계의 시간 보정에 영향을 받지 않는다

### wait와 notify의 올바른 사용방법
- 생략