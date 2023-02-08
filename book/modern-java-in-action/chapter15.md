# 15 CompletableFuture와 리액티브 프로그래밍 기초
- 소프트웨어 개발 방법을 획기적으로 뒤집는 두 가지 추세가 있다
  - 멀티코어 프로세서의 발전
    - 한 개의 큰 태스크를 개별 하위 태스크로 나누어 병렬 실행할 수 있다
    - 포크/조인 프레임 워크, 병렬 스트림으로 스레드에 비해 단순하고 효과적인 방법으로 병렬 실행을 달성할 수 있다
  - 인터넷 서비스에서 사용하는 애플리케이션이 증가
    - 마이크로 서비스 아키텍처
      - 서비스가 작아지는 대신 네트워크 통신이 늘어났다
    - 웹 애플리케이션은 앞으로 다양한 소스의 콘텐츠를 가져와서 사용자가 삶을 풍요롭게 만들도록 합치는 매시업 형태가 될 가능성이 높다
- 외부 서비스를 호출하고난 후 응답을 기다리는 동안 연산이 블록 되거나 CPU 클록 사이클 자원을 낭비하고 싶지 않을 수 있다
  - 이러한 상황은 병렬성의 양면성을 보여준다
- 병려성이 아닌 동시성을 필요로 하는 상황이 있을 수 있다
- 자바는 이러한 상황에서 사용할 수 있는 중요한 두 가지 도구를 제공한다
  - CompletableFuture
  - 플로 API

- 동시성은 단일 코어 머신에서 발생할 수 있는 프로그래밍 속성으로 실행이 서로 겹칠 수 있다
- 병렬성은 병렬 실행을 하드웨어 수준에 지원한다

## 15.1 동시성을 구현하는 자바 지원의 진화
- 처음 자바는 `Runnable`과 `Thread`를 동기화된 클래스와 메서드를 이용해 잠갔다
- 자바 5에서는 스레드 실행과 태스크 제출을 분리하는 `ExcutorService` 인터페이스, `Callable<T>`과 `Future<T>`, 제네릭등을 지원했다
- 자바 7에서는 분할-정복 알고리즘의 포크/조인 구현을 지원하는 `java.util.concurrent.RecursiveTask`가 추가되었다
- 자바 8에서는 스트림과 새로 추가된 람다 지원에 기반한 병렬 프로세싱이 추가되었다
- 자바 9에서는 분산 비동기 프로그래밍을 명시적으로 지원한다.(발행-구독 프로토콜)
- 이들 API는 다양한 웹 서비스를 이용해 정보를 실시간으로 조합하여 사용자에게 제공하거나 추가 웹 애플리케이션을 개발하는데 필수적인 기초 모델과 툴킷을 제공한다
  - 리액티브 프로그래밍이라 한다

- `CompletableFuture`와 `java.util.concurrent.Flow`의 궁극적인 목표는 가능한한 동시에 실행할 수 있는 독립적인 태스크를 가능하게 만들면서 멀티코어 혹은 여러 기기를 통해 제공되는 병렬성을 쉽게 이용하는 것이다

### 15.1.1 스레드와 높은 수준의 추상화
- 단일 CPU 컴퓨터도 여러 사용자를 지원한다
  - 운영체제가 각 사용자에게 프로세스 하나를 할당한다
  - 둔영체제는 각 사용자가 독립된 공간을 가질 수 있도록 가상 주소 공간을 각각의 프로세스에 제공한다
  - 운영체제는 각 프로세스를 번갈아가며 실행한다
  - 프로세스는 운영체제에게 스레드(본인이 가진 같은 주소 공간을 공유하는 프로세스)를 요청하여 태스크를 동시에 또는 협력적으로 실행할 수 있다

- 멀티코어에서는 스레드의 도움 없이 프로그램이 컴퓨팅 파워를 모두 활용할 수 없다
  - 각 코어는 한 개 이상의 프로세스나 스레드에 할당될 수 있지만, 프로그램이 스레드를 사용하지 않는다면 효율성을 고려해 하나의 코어만 사용할 것이다
- 자바의 병렬 스트림을 사용하면 스레드 사용 패턴을 추상화할 수 있다

### 15.1.2 Executor와 스레드 풀

**스레드의 문제**
- 자바 스레드는 운영체제에 직접 접근한다
  - 운영체제 스레드는 비용이 크다
  - 운영체제 스레드는 개수가 제한되어 있다
- 운영체제와 자바의 스레드 개수가 하드웨어 스레드 개수보다 많아 일부 운영체제 스레드가 블록된 상황에서 모든 하드웨어 스레드가 코드를 실행하도록 할당된 상황이 생길 수 있다
  - 운영체제, 자바 스레드 개수 > 하드웨어 스레드 개수
  - 일반적인 서버와 노트북은 하드웨서 스레드 개수가 다르므로 여러 환경에서 실행될 수 있는 프로그램은 하드웨어 스레드 개수를 예측하지 않는 것이 좋다

**스레드 풀 그리고 스레드 풀이 더 좋은 이유**
- 자바 `ExecutorService`는 태스크를 제출하고 나중에 결과를 수집할 수 있는 인터페이스를 제공한다
- `newFixedThreadPool` 같은 팩토리 메서드를 이용해 스레드 풀을 만들어 사용할 수 있다
  - 워커 스레드를 만들어 스레드 풀에 저장한다
  - 태스크를 먼저 온 순서대로 처리한다
- 하드웨어에 맞는 수의 태스크를 유지함과 동시에 수 천개의 태스크를 스레드 풀에 아무 오버헤드 없이 제출할 수 있다
- 프로그래머는 태스크(Runnable, Callable)를 제공하고, 스레드는 이를 실행한다

**스레드 풀 그리고 스레드 풀이 나쁜 이유**
- k 스레드를 가진 스레드 풀은 오직 k 만큼의 스레드를 동시에 실행할 수 있다
  - 초과로 제출된 태스크는 큐에 저장되어 대기한다
  - 실행중인 태스크가 잠을 자거나 I/O를 기다리면 작업의 효율성이 떨어진다
  - 처음 제출한 태스크가 나중에 제출한 태스크를 기다리는 상황이면 데드락이 발생할 수 있다
  - 블록할 수 있는 태스크는 제출하지 말아야하겠지만 쉽지않다

- 자바 프로그램은 `main`이 반환하기 전에 모든 스레드의 작업이 끝나길 기다린다
  - 프로그램을 종료하기 전에 모든 스레드 풀을 종료하는 습관을 가지는 것이 좋다

### 15.1.3 스레드의 다른 추상화: 중첩되지 않는 메서드 호출
- 엄격한 포크/조인
  - 태스크나 스레드가 메서드 호출 안에서 시작되면 그 메서드 호출은 반환하지 않고 작업이 끝나기를 기다린다
  - 다시 말해 스레드 생성과 `join()`이 한 쌍처럼 중첩된 메서드 호출 내에 추가된다
- 여유로운 포크/조인
  - 시작된 태스크를 내부 호출이 아니라 외부 호출에서 종료하도록 기다린다
  - 호출한 메서드를 벗어나 계속 실행된다

![Screen Shot 2022-07-28 at 7 22 59 AM](https://user-images.githubusercontent.com/60502370/181382809-ff35b1b8-2c7d-4c36-905c-8a106e166098.png)

- 메서드 호출자에 기능을 제공하도록 메서드가 반환된 후에서 만들어진 태스크 실행이 계속되는 메서드를 비동기 메서드라 한다

![image](https://user-images.githubusercontent.com/60502370/181382993-3f852903-2651-4234-b089-6d58754349e1.png)

- 비동기 메서드의 위험성
  - 스레드 실행은 메서드를 호출한 다음의 코드와 동시에 실행되므로 데이터 경쟁 문제를 일으키지 않도록 주의해야 한다
  - 기존 실행 중이던 스레드가 종료되지 않은 상황에서 자바의 `main()` 메서드가 반환되면 안전하지 못하다.
    - 불안전한 해결책 두 가지
      - 애플리케이션이 종료하지 못하고 모든 스레드가 실행을 끝낼 때까지 기다린다
        - 잊고서 종료를 못한 스레드에 의해 애플리케이션이 크래시될 수 있다
      - 애플리케이션 종료를 방해하는 스레드를 강제 종료시키고 애플리케이션을 종료한다
        - 디스크에 I/O 작업을 시도하는 작업이 중단되었을 때 외부 데이터의 일관성이 파괴될 수 있다
- 자바 스레드는 `setDaemon()` 메서드를 이용해 데몬 혹은 비데몬으로 구분할 수 있다
  - 데몬 스레드는 애플리케이션이 종료될 때 강제 종료되므로 데이터 일관성을 파괴하지 않는 경우 유용하게 사용할 수 있다
  - `main()`은 비데몬 스레드가 종료될 때까지 기다린다

### 15.1.4 스레드에 무엇을 바라는가?
- 프로그램을 작은 태스크 단위로 구조화하는 것이 목표다
  - 모든 하드웨어 스레드를 활용해 병렬성의 장점을 극대화 할 수 있다

## 15.2 동기 API와 비동기 API
- 스트림에 `parallel()` 메서드를 이용하면 자바 런타임 라이브러리가 복잡한 스레드 작업을 하지 않고 병렬로 요소가 처리되도록 할 수 있다
- 루프 기반의 계산을 제외한 다른 상황에서도 병렬성이 유용할 수 있다

**예제**
- 다음과 같은 시그니처를 가지는 f, g 두 함수가 있다고 가정한다
```java
int f(int x);
int g(int x);
```

- 두 함수의 결과를 합하는 작업을 수행한다
- f, g의 작업 시간이 오래걸리고 작업이 복잡해 컴파일러가 최적화할 수 없다
- 별도의 스레드로 f, g를 실행할 수 있지만, 구조가 복잡해진다

```java
class ThreadExample {
    public static void main(String[] args) throws InterruptedException {
        int x = 1337;
        Result result = new Result();

        Thread t1 = new Thread(() -> {result.left = f(x);});
        Thread t2 = new Thread(() -> {result.right = g(x);});

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println(result.left + result.right);
    }

    private static class Result {
        private int left;
        private int right;
    }
}
```

- `Runnalbe`대신 `Future`를 사용해 코드를 더 단순화 할 수 있다

```java
class ExecutorServiceExample {
    public static void main(String[] args) throws InterruptedException {
        int x = 1337;

       ExecutorService executorService = Executors.newFixedThreadPool(2);
       Future<Integer> y = executorService.submit(() -> f(x));
       Future<Integer> z = executorService.submit(() -> g(x));

       System.out.println(y.get() + z.get());
       executorService.shutdown();
    }
}
```

- `submit` 메서드 호출 같은 불필요한 코드로 오염되어 있다
- 명시적 방복으로 병렬화를 수행하던 코드를 스트림을 이용해 내부 반복으로 바꾼것처럼 비슷한 방법으로 이 문제를 해결해야 한다
- 비동기 API로 해결할 수 있다
  - 자바 5에서 소개된 `Future`는 자바 8의 `CompletableFuture`로 이들을 조합해서 더욱 기능이 풍부해졌다
  - 자바 9의 발행-구독 프로토콜에 기반한 `java.util.concurrent.Flow` 인터페이스를 이용할 수 있다


### 15.2.1 Future 형식 API
- 함수 시그니처가 아래와 같이 바뀐다

```java
Future<Integer> f(int x);
Future<Integer> g(int x);
```

- 호출도 아래와 같이 바뀐다

```java
Future<Integer> y = f(x);
Future<Integer> z = g(x);

System.out.println(y.get() + z.get());
```

- f, g 메서드는 호출 즉시 자신의 원래 바디를 평가하는 태스크를 포함하는 `Future`를 반환한다
- `get()`은 두 `Future`가 완료되어 결과가 합쳐지기를 기다린다
- 예제에서는 API는 그대로 유지하고 g를 그대로 호출하면서 f에만 `Future`를 적용할 수 있었다. 하지만 조금 더 큰 프로그램에서는 다음과 같은 두 가지 이유로 이러한 방식을 사용하지 않는다
  - 다른 상황에서는 g도 `Future` 형식이 필요할 수 있으므로 API 형식을 통일하는 것이 바람직하다(?)
  - 병렬 하드웨어로 프로그램 실행 속도를 극대화하려면 여러 작은 하지만 합리적인 크기로 태스크를 나누는 것이 좋다

### 15.2.2 리액티브 형식 API
- 두 번째 대안의 핵심은 f, g의 시그니쳐를 바꿔서 콜백 형식을 프로그래밍을 사용하는 것이다

```java
void f(int x, IntConsumer dealWithResult);
```

- f에 추가 인수로 콜백을 전달해서 f의 바디에서는 return 문으로 결과를 반환하는 것이 아니라 결과가 준비되면 이를 람다로 호출하는 태스크를 만든다
  - f는 바디를 실행하면서 태스크를 만든 다음 즉시 반환하므로 다음과 같이 코드가 바뀐다

```java
public class CallbackStyleExample {
    public static void main(String[] args) {
        int x = 1337;
        Result result = new Result();

        f(x, (int y) -> {
            result.left = y;
            System.out.println((result.left + result.right));
        });


        g(x, (int z) -> {
            result.right = z;
            System.out.println((result.left + result.right));
        });
    }
}
```

- 하지만 f와 g의 호출 합계를 정확하게 출력하지 않고 상황에 따라 먼저 계산된 결과를 출력한다
- 락을 사용하지 않으므로 값을 두 번 출력할 수 있고 때로는 `+`에 제공된 두 피연산자가 `println`이 호출되기 전에 업데이트될 수 있다
- 두 가지 해결책
  - `if-then-else`를 이용해 적절한 락을 사용하여 두 콜백이 모두 호출되었는지 확인한 다음 `println`을 호출한다
  - 리액티브 형식의 API는 보통 한 결과가 아니라 일련의 이벤트에 반응하도록 설계되었으므로 `Future`를 이용하는 것이 더 적절하다

- 리액티브 형식의 비동기 API는 자연스럽게 일련의 값을, Future 형식의 API는 일회성의 값을 처리하는데 적합하다

### 15.2.3 잠자기(그리고 기타 블로킹 동작)는 해로운 것으로 간주
- 사람과 상호작용하거나 어떤 일이 일정 속도로 제한되어 일어나는 상황의 애플리케이션을 만들때 `sleep()` 메서드를 사용할 수 있다 
  - 스레드는 잠들어도 시스템 자원을 점유한다
  - 스레드 풀에서 잠자는 태스크는 다른 태스크가 시작되지 못하게 막으므로 자원을 소비한다
- 스레드 풀에서 잠자는 스레드 뿐 아니라 모든 블록 동작도 자원을 소비한다
  - 블록은 다른 태스크가 어떤 동작을 완료하기를 기다리는 동작(`Future.get()` 등), 외부 상호작용(네트워크, 사용자 입력 대기) 두 가지로 나눌 수 있다
- 이상적으로는 이러한 상황을 막기위해 태스크에서 기다리는 일을 만들지 않거나 코드에서 예외를 일으키는 방법으로 처리할 수 있다
- 태스크를 앞과 뒤 부분으로 나누고 블록되지 않을 때만 뒷 부분을 자바가 스케줄링하도록 요청할 수 있다

**코드A**

```java
work1();
Thread.sleep(1000);
work2();
```
- `work1` 실행 후 스레드를 점유한 상태에서 10초를 잔다
- 그리고 꺠어나서 `work2`를 실행한다음 작업을 종료하고 워커 스레드를 해제한다

**코드B**
```java
public class ScheduledExecutorServiceExample {
    public static void main(String[] args) {
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);

        work1();
        scheduledExecutorService.schedule(
            ScheduledExecutorServiceExample::work2, 10, TimeUnit.SECONDS
        ); // work1이 끝난 다음 10초 뒤에 work2를 개별 태스크로 스케줄함

        scheduledExecutorService.shutdown();
    }
}
```
- `work1`을 실행하고 종료한다. 10초 뒤에 `work2`가 실행될 수 있도록 큐에 추가한다
- 10초 뒤 `work2`가 실행된다
- 코드A와 다르게 10초 동안 스레드를 점유하지 않는다

- 가능하면 I/O 작업에서도 코드B와 같은 원칙을 적용하는 것이 좋다
  - 고전적으로 읽기 작업을 기다리는 것이 아니라, 읽기 시작 메서드를 호출하고 읽기 작업이 끝나면 이를 처리할 다음 태스크를 런타임 라이브러리에 스케줄하도록 요청하고 종료한다
- 자바 `CompletableFuture` 인터페이스는 이전에 살펴본 `Future`에 `get()`을 이용해 명시적으로 블록하지 않고 콤비테이터를 사용함으로 이런 형식의 코드를 런타임 라이브러리 내에 추상화한다

### 15.2.4 현실성 확인
- 모든 동작을 비동기 호출로 구현한다면 병렬 하드웨어를 최대한 활용할 수 있다
- 하지만 현실적으로 모든 것은 비동기라는 설계 원칙을 어겨야 한다
- 실제로 자바의 개선된 동시성 API를 이용해 유익을 얻을 수 있는 상황을 찾아보고 모든 API를 비동기로 만드는 것을 따지지 말고 개선된 동시성 API를 사용해 보길 권장한다


### 15.2.5 비동기 API에서 예외는 어떻게 처리하는가?
- `Future`나 리액티브 형식의 비동기 API에서 호출된 메서드의 실제 바디는 별도의 스레드에서 호출되며 이때 발생하는 어떤 에러는 이미 호출자의 실행 범위와는 관계가 없는 상황이 된다
- `CompletableFuture`에서는 런타임 `get()` 메서드에서 예외를 처리할 수 있는 기능을 제공하며 예외에서 회복할 수 있도록 `exceptionally()` 같은 메서드도 제공한다
- 비동기 API에서는 return 대신 기존 콜백이 호출되므로 예외가 발생했을 때 실행될 추가 콜백을 만들어 인터페이스를 바꿔야 한다

```java
void f(int x, Consumer<Integer> dealWithResult, Consumer<Integer> dealWithException);
```

- 콜백이 여러 개면 이를 따로 제공하는 것보다는 한 객체로 이 메서드를 감싸는 것이 좋다
  - 자바 9 Flow API에서는 여러 콜백을 한 객체로 감싼다.(`Subscriber<Integer> s`)

```java
void onComplete();
void onError(Throwable throwable);
void onNext(T item);
```
- complete, error, next 는 어떠한 일이 발생했음을 말하는 것인데 이를 이벤트라 한다

## 15.3 박스와 채널 모델
- 박스와 채널 모델은 동시성 모델을 잘 설계화하고 개념화하기 위한 기법이다
- p 함수를 호출한 결과를 q1, q2에 전달하고 그 결과를 다시 r 함수에 전달한 결과를 출력하는 태스크를 아래와 같은 그림으로 표현할 수 있다

<img width="381" alt="Screen Shot 2022-07-31 at 2 27 47 PM" src="https://user-images.githubusercontent.com/60502370/182011627-1f38fedf-7c1d-4c56-a386-eb93c0ee2c96.png">

- 태스크를 두 가지 방법으로 구현할 수 있다
**방법 1**

```java
int t = p(x);
System.out.println(r(q1(t), q2(t)));
```
- 겉보기엔 깔끔해 보이는 코드이다
- 하지만 자바가 q1, q2를 차례로 호출하는데 이는 하드웨어 병렬성의 활용과 거리가 멀다

**방법 2**
- `Future`를 사용해 q1, q2를 병렬로 평가하는 방법도 있다

```java
int t = p(x);
Future<Integer> a1 = executorService.submit(() -> q1(t));
Future<Integer> a2 = executorService.submit(() -> q2(t));
System.out.println(r(a1.get(), a2.get()));
```
- 이 예제에서는 박스와 채널 다이어그램의 모양상 p와 r을 Future로 감싸지 않았다
  - p는 다른 어떤 작업보다 먼저 실행해야 하고, r은 가장 마지막에 실행해야 한다
- 아래처럼 코드를 흉내내보지만 이는 우리가 원하는 작업과 거리가 있다(?)
  - `System.out.println(r(q1(t), q2(t)) + s(x));`
  - 위 코드에서 병렬성을 극대화 하려면 모든 다섯 함수(p, q1, q2, r, s)를 Future로 감싸야하기 떄문이다
  - s(x)는 뭐고, 원하는 작업은 무엇인가?

- 시스템에서 많은 작업을 동시에 실행하지 않고 있다면 모든 함수를 `Future`로 감싸는 것도 잘동작할 수 있다
- 하지만 시스템이 커지고 각각의 박스와 채널다이어 그램이 등장하고 각각의 박스가 내부적으로 자신만의 박스와 채널을 사용하면 문제가 달라진다
  - 많은 태스크가 `get()` 메서드를 호출해 `Future`가 끝나기를 기다리는 상태에 놓일 수 있다
  - 병렬성을 제대로 활용하지 못하거나 데드락에 걸릴 수 있다
  - 대규모 시스템에서 얼마나 많은 `get()`을 감당할 수 있을지 알기 어렵다
- 이러한 문제를 `CompletableFuture`와 콤비네이터를 이용해 해결할 수 있다
- 박스와 채널 모델을 이용해 **생각과 코드를 구조화할 수 있다**
  - 대규모 시스템 구현의 추상화 수준을 높일 수 있다
  - 박스로 원하는 연산을 표현하면 계산을 손으로 코딩한 것보다 효율적이다
  - 콤비네이터는 수학적 함수 뿐 아니라 `Future`와 리액티브 스트림 데이터에 적용할 수 있다
  - 박스와 채널 모델은 병렬성을 직접 프로그래밍하는 관점을 콤비네이터를 이용해 내부적으로 작업을 처리하는 관점으로 바꿔준다

## 15.4 CompletableFuture와 콤비네이터를 이용한 동시성
- 자바 8에서는 `Future` 인터페이스의 구현인 `CompletableFuture`를 이용해 `Future`를 조합할 수 있는 기능을 추가했다
- `CompletableFuture`라 이름 지은 이유는 다음과 같다
  - `Future`는 실행해서 `get()`의 결과를 얻을 수 있는 Callable로 만들어진다
  - 하지만 `CompletableFuture`는 실행할 코드 없이 `Future`를 만들도록 허용한다
  - 그리고 `complete()` 메서드를 이용해 나중에 어떤 값을 이용해 다른 스레드가 이를 완료할 수 있고 `get()`으로 값을 얻을 수 있도록 허용한다

```java
public class CFComplete {
  public static void main(String[] args) throws ExecutionException, InterruptedException {
      ExecutorService executorService = Executors.newFixedThreadPool(10);
      int x = 1337;

      CompletableFuture<Integer> a = new CompletableFuture<>();
      executorService.submit(() -> a.complete(f(x))); // executorService.submit(() -> a.complete(g(x)));
      int b = g(x); // f(x);
      System.out.println(a.get() + b); // System.out.println(a + b.get());

      executorService.shutdown();
  }
}
```

- 위 코드는 `f(x)` 혹은 `g(x)`의 실행이 끝나지 않는 상황에서 `get()`을 기다려야 하므로 프로세싱 자원을 낭비할 수 있다
- `f(x)`와 `g(x)`를 실행하는 태스크와 합계를 계산하는 세 번째 태스크를 이용하면 문제를 해결할 수 있다
- 하지만 합계를 구하는 태스크는 다른 두 태스크가 실행한 후 마지막에 실행해야만 한다
- `CompletableFuture<T>`에 `thenCombine` 메서드를 사용함으로 두 연산의 결과를 더 효과적으로 더할 수 있다
  - `CompletableFuture<V> thenCombine(CompletableFuture<U> other, BiFunction<T, U, V> fn)`

```java
public class CFComplete {
  public static void main(String[] args) throws ExecutionException, InterruptedException {
      ExecutorService executorService = Executors.newFixedThreadPool(10);
      int x = 1337;

      CompletableFuture<Integer> a = new CompletableFuture<>();
      CompletableFuture<Integer> b = new CompletableFuture<>();
      CompletableFuture<Integer> c = a.thenCombine(b, (y, z) -> y + z);
      executorService.submit(() -> a.complete(f(x)));
      executorService.submit(() -> b.complete(g(x)));

      System.out.println(c.get());
      executorService.shutdown();
  }
}
```

- `thenCombine`행이 핵심이다 
  - a와 b의 결과를 알지 못한 상태에서 `thenCombine`은 두 연산이 끝났을 때 스레드 풀에서 실행된 연산을 만든다
  - c 연산은 다른 두 작업이 끝날 때까지는 스레드에서 실행되지 않는다(먼저 시작되서 블록이 되는 것 아님)
- 기존의 코드에서 발생헀던 블록 문제가 해결되었다

## 15.5 발행-구독 그리고 리액티브 프로그래밍
- `Future`는 한 번만 실행해 결과를 제공한다
- 리액티브 프로그래밍은 시간이 흐르면서 여러 `Future` 같은 객체를 통해 여러 결과를 제공한다
  - 온도계 객체는 매 초마다 온도 값을 반복적으로 제공한다
  - 웹 서버 컴포넌트 응답을 기다리는 리스너 객체는 네트워크에서 HTTP 요청이 발생하길 기다렸다가 이후에 결과 데이터를 생산한다
- 리액티브 프로그래밍이라 불리는 이유는 특정 이벤트에 반응(React)하기 떄문이다
  - 온도가 낮아지면 히터를 킨다
- 자바 9에서는 `java.util.concurrent.Flow`의 인터페이스에 발행-구독 모델을 적용해 리액티브 프로그래밍을 제공한다
- 플로 API는 다음처럼 세 가지로 정리할 수 있다
  - 구독자가 구독할 수 있는 발행자
  - 이 연결을 구독(Subscription)이라 한다
  - 이 연결을 이용해 메시지(또는 이벤트로 알려짐)를 전송한다

### 15.5.1 두 플로를 합치는 예제
- 스프레드시트 예제
- '=C1+C2'라는 공식을 포함하는 스프레드시트 셀 C3 구현

```java
public class SimpleCell {
  private int value = 0;
  private String name;
  
  public SimpleCell(String name) {
    this.name = name;
  }
}
```

- 다음과 같이 셀을 생성한다

```java
SimpleCell c2 = new SimpleCell("c2");
SimpleCell c1 = new SimpleCell("c1");
```

- c1와 c2에 이벤트가 발생했을 때 c3를 구독하도록 만든다

```java
interface Publisher<T> {
  void subscribe(Subscriber<? super T> subscriber);
}
```
- 이 인터페이스는 통신할 구독자를 인수로 받는다

```java
interface Subscriber<T> {
  void onNext(T t);
}
```
- 이 인터페이스는 `onNext`라는 정보를 전달할 단순 메서드를 포함한다
- Cell은 Publisher임과 동시에 Subscriber이다

```java
public class SimpleCell implements Publisher<Integer>, Subscriber<Integer> {
  private int value = 0;
  private String name;
  private List<Subscriber> subscribers = new ArrayList<>();

  public SimpleCell(String name) {
    this.name = name;
  }

  @Override
  public void subscribe(Subscriber<? super Integer> subscriber) {
    subscribers.add(subscriber);
  }

  private void notifyAllSubscribers() {
    subscribers.forEach(subscriber -> subscriber.onNext(this.value)); //새로운 값이 있음을 모든 구독자에게 알림
  }

  @Override
  public void onNext(Integer newValue) {
    this.value = newValue;
    System.out.println(this.name + " :" + this.value); // 값을 콘솔로 출력(UI)로 변경 가능
    notifyAllSubscribers(); //값이 갱신되었음을 모든 구독자에게 알림
  }
}
```
- 간단한 예제 실행 결과는 아래와 같다


```java
SimpleCell c3 = new SimpleCell("c3");
SimpleCell c2 = new SimpleCell("c2");
SimpleCell c1 = new SimpleCell("c1");

c1.subscribe(c3);

c1.onNext(10);
c2.onNext(20);

/**
C1:10
C3:10 // C1의 값을 10으로 갱신
C2:20
*/
```

- 'C3=C1+C2'를 구현하기 위해서는 왼쪽과 오른쪽 연산 결과를 저장할 수 있는 별도의 클래스가 필요하다

```java
public class ArithmeticCell extends SimpleCell {
  private int left;
  private int right;

  public ArithmeticCell(String name) {
    super(name);
  }

  public void setLeft(int left) {
    this.left = left;
    onNext(left + this.right);
  }

  public void setRight(int right) {
    this.right = right;
    onNext(right + this.left);
  }
}
```
- 간단한 예제 실행 결과는 아래와 같다


```java
ArithmeticCell c3 = new ArithmeticCell("c3");
SimpleCell c2 = new SimpleCell("c2");
SimpleCell c1 = new SimpleCell("c1");

/* 
* onNext 구현
c1.subscribe(left -> c3.setLeft(left));
c2.subscribe(right -> c3.setRight(right));
*/
c1.subscribe(c3::setLeft);
c2.subscribe(c3::setRight);

c1.onNext(10);
c2.onNext(20);
c1.onNext(15);

/**
C1:10
C3:10
C2:20
C3:30
C1:15
C3:35
*/
```

- 발행자-구독자 상호작용의 멋진 점은 발행자 구독자의 그래프를 설정할 수 있다는 것이다
  - 'C5=C3+C4'처럼 C3과 C4에 의존하는 새로운 셀 C5를 만들 수 있다
- 간단하지만 플로 인터페이스의 개념을 복잡하게 만든 두 가지 기능은 압력과 역압력이다
  - 모든 SMS 메시지를 폰으로 제공하는 발행자에 가입하는 상황을 가정하자. 처음에는 약간의 SMS 메시지가 있는 새 폰에서는 가입이 잘 동작할 수 있지만 몇 년 후에는 매 초마다 수천 개의 메시지가 onNext로 전달된다면 어떤일이 발생할까? 이러한 상황을 **압력**이라 한다
  - 공에 담긴 메시지를 포함하는 수직 파이프를 상상해보자. 이런 상황에서는 출구로 추가될 공의 숫자를 제한하는 역압력 같은 기법이 필요하다. 자바 9 플로 API에서는 발행자가 무한의 속도로 아이템을 방출하는 대신 요청했을 때만 다음 아이템을 보내도록 하는 `request()` 메서드를 제공한다

### 15.5.2 역압력
- 정보의 흐름 속도를 역압력(흐름 제어)으로 제어 할 필요가 있을 수 있다
  - Subscriber에서 Publisher로 정보를 요청해아 할 필요가 있을 수 있다
  - 정보의 흐름이 원래는 Subscriber -> Publisher 였는데 Subscriber <- Publisher로 뒤바뀔 필요가 있을 수 있다
- Publisher는 여러 Subscriber를 갖고 있으므로 역압력 요청이 한 연결에만 영향을 미쳐야한다
- 자바 9의 플로 API의 Subscriber 인터페이스는 `onSubscribe(Subscription subscription)` 메서드를 포함한다
- `Subscription` 객체는 다음처럼 Subscriber와 Publisher와 통신할 수 있는 메서드를 가진다

```java
interface Subscription {
  void cancel();
  void request(long n);
}
```

- Publisher는 Subscription 객체를 만들어 Subscriber에 전달하면 Subscriber는 이를 이용해 Publisher로 전도를 보낼 수 있다

### 15.5.3 실제 역압력의 간단한 형태
- 한 번에 한 개의 이벤트를 처리하도록 발행-구독 연결을 구성하려면 다음과 같은 작업이 필요하다
  - Subscriber가 OnSubscribe로 전달된 Subscription 객체를 subscription 같은 필드에 로컬로 저장한다
  - Subscriber가 수 많은 이벤트를 받지 않도록 onSubscribe, onNext, onError의 마지막 동작에 channel.request(1)을 추가해 오직 한 이벤트만 요청한다
  - 요청을 보낸 채널에만 onNext, onError 이벤트를 보내도록 Publisher의 notifyAllSubscribers 코드를 바꾼다
- 구현이 간단해 보일 수 있지만, 역압력을 구현하려면 여러가지 장단점을 생각해야 한다
  - 여러 Subscriber가 있을 때 이벤트를 가장 느린 속도록 보낼 것인가? 아니면 각 Subscriber에게 보내지 않은 데이터를 저장할 별도의 큐를 가질 것인가?
  - 큐가 너무 커지면 어떻게 할 것인가?
  - Subscriber가 준비가 안됐다면 큐의 데이터를 폐기할 것인가?

- 당김 기반 리액티브 역압력은 Subscriber가 Publisher로부터 요청을 당긴다

## 15.6 리액티브 시스템 VS 리액티브 프로그래밍
- 리액티브 시스템
  - 런타임 환경 변화에 대응하도록 전체 아키텍처가 설계된 프로그램
  - 반응성, 회복성, 탄력성을 가져야 한다
  - 반응성이란 시스템이 큰 작업을 처리하느라 간단한 질의의 응답을 지연하지 않고 실시간으로 입력에 반응하는 것을 말한다
  - 회복성은 한 컨포넌트의 실패로 전체 시스템이 실패하지 않음을 의미한다
  - 탄력성은 시스템이 자신의 작업 부하에 맞게 적응하며 작업을 효율적으로 처리함을 의미한다
- 리액티브 프로그래밍
  - 리액티브 시스템을 구현할 수 있는 프로그래밍 기법이다