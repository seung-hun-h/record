# 병렬 데이터 처리와 성능
## 병렬 스트림
-  `parallelStream`을 호출하면 병렬 스트림이 생성된다
- 병렬 스트림이란 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림이다

### 순차 스트림을 병렬 스트림으로 변환하기
- 순차 스트림에 `parallel` 메서드를 호출하면 기존의 함수형 리듀싱 연산이 병렬로 처리된다
- `parallel`을 호출하면 스트림 자체에는 변화가 없고, 연산이 병렬로 처리되야 함을 의미하는 불리은 플래그가 설정된다
- `sequential`로 병렬 스트림을 순차 스트림으로 변경할 수 있다
- `parallel`과 `sequential` 중 최종적으로 호출된 메서드가 전체 파이프라인에 영향을 미친다

### 병렬 스트림에서 사용하는 스레드 풀 설정
- 병렬 스트림은 내부적으로 ForkJoinPool을 사용한다
- ForkJoinPool은 기본적으로 `Runtime.getRuntime().availableProcessors()`가 반환하는 값에 상응하는 스레드를 갖는다
  - 프로세서의 수와 상응하는 스레드를 가진다

### 스트림 성능 측정
- `iterate`를 사용해 합계 연산을 수행할 때, 병렬 스트림을 사용하면 오히려 외부 반복을 사용한 합계 연산보다 성능이 떨어진다
  - 이는 스트림을 여러개의 청크로 분할할 수 없기 때문이다
  - `iterate`는 리듀싱 과정을 시작하는 시점에서 숫자 리스트가 준비되어 있지 않기 때문에 병렬로 처리할 수 있도록 청크로 분할하기 어렵다

**특화된 메서드 사용**
- `LongStream.rangeClosed`는 `iterate`보다 다음과 같은 장점을 가진다
  - 기본형 long을 사용하므로 박싱과 언박싱 오버헤드가 사라진다
  - 쉽게 청크로 분할할 수 있는 숫자 범위를 생산한다

- 병렬화를 이용하기 위해서는 스트림을 재귀적으로 분할해야 하고, 각 서브 스트림을 서로 다른 스레드의 리듀싱 연산으로 할당하고, 이들 결과를 하나의 값으로 합쳐야 한다
  - 멀티코어 간의 데이터 이동의 오버헤드는 크다
  - 코어 간에 데이터 전송 시간보다 훨씬 오래 걸리는 작업만 병렬로 다른 코어에서 수행하는 것이 바람직하다

### 병렬 스트림의 올바른 사용법
- 공유된 상태를 바꾸는 알고리즘의 경우 병렬 스트림을 사용하면 문제가 발생한다

```java
public long sideEffectSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).forEach(accumulator::add);
    return accumulator.total;
}

public class Accumulator {
    public long total = 0;
    public void add(long value) { total += value; }
}
```
- 위 코드에서 `Accumulator.total`은 공유 데이터이기 떄문에 올바른 결과를 가지기 어렵다

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/170382177-b2a32403-a46e-439b-93f5-48ab1dc62a43.png">

- 여러 스레드에서 동시에 `total += value`를 실행하면서 문제가 발생한다
  - 위 연산은 아토믹 연산이 아니다
  - 문제를 해결하기 위해서는 공유된 가변 상태를 피해야 한다

### 병렬 스트림 효과적으로 사용하기
- 양을 기준으로 병렬 스트림 사용을 결정하는 것은 적절하지 않다
- 확신이 서지 않으면 측정하라
  - 무조건 병렬 스트림으로 바꾸는 것이 능사는 아니다
  - 순차, 병렬 중 어떤 것이 좋을 지 모르겠다면 직접 성능을 측정하는 것이 바람직하다
- 박싱을 주의하라
  - 자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있는 요소다
  - 기본형 특화 스트림을 사용하라
- 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있다
  - `limit`, `findFirst`처럼 요소 순서에 의존하는 연산은 병렬 스트림에서 오버헤드가 크다
  - 순서가 상관없는 `findAny` 혹은 정렬된 스트림에 `unordered`를 사용하는 것이 좋다
- 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라
  - 처리해야 할 요소 수가 N, 하나의 요소를 처리하는데 드는 비용이  Q일때
  - 전체 스트림 파이프라인 처리 비용을 N * Q로 예상할 수 있다
  - Q가 높아진다는 것은 병렬 스트림으로 성능을 개선할 수 있다
- 소량의 데이터에서는 병렬 스트림이 도움되지 않는다
- 스트림을 구성하는 자료구조가 적절한지 확인하라
  - ArrayList를 LinkedList보다 효율적으로 분할할 수 있다
  - `range` 팩토리 메서드로 만든 기본형 스트림도 쉽게 분할할 수 있다
- 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다
  - 필터 연산이 있으면 스트림의 길이를 예측할 수 없으므로 효과적으로 스트림을 처리할 수 있을 지 알 수 없게 된다
- 최종 연산의 병학 과정 비용을 살펴보라
  - 병합 과정의 비용이 비싸다면 병렬 스트림으로 얻는 성능의 이익이 서브 스트림의 부분 결과를 합치는 과정에서 상쇄될 수 있다

## 포크/조인 프레임워크
- 포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 잡업으로 분할한 다음에 서브태스크 각각의 결과를 합쳐서 전체 결과를 만들도록 설계되었다
- 포크/조인 프레임워크에서는 서브태스크를 스레드 풀(ForkJoinPool)의 작업자 스레드에 분산 할당하는 ExecutorService 인터페이스를 구현한다

### RecursiveTask 활용
- 스레드 풀을 이용하려면 `RecursiveTask<R>`의 서브 클래스를 만들어야 한다
  - `R`은 병렬화된 태스크가 생성하는 결과 형식 또는 결과가 없을 때는 `RecursiveAction` 형식(타입)이다
- `RecursiveTask`를 정의하려면 `compute`를 구현해야 한다
  - `protected abstract R compute();`
  - `compute`는 태스크를 서브태스크로 분할하는 로직과 더 이상 분할할 수 없을 때 개별 서브태스크의 결과를 생산할 알고리즘을 정의한다
```txt
- compute 의사 코드- 

if (태스크가 충분히 작거나 더 이상 분할할 수 없는 경우) {
  순차적으로 태스크 계산
} else {
  태스크를 두 서브태스크로 분할
  태스크가 다시 서브태스크로 분할되도록 이 메서드를 재귀적으로 호출
  모든 서브태스크의 연산이 완료될 때까지 기다림
  각 서브태스크의 결과를 합침
}

```

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/170605917-59bc8040-5f0d-4b96-8f37-b24ab9190730.png">

```java
public class ForkJoinSumCalculator extends java.util.concurrent.RecursiveTask<Long> {
  private final long[] numbers;
  private final int start;
  private final int end;
  private static final long THRESHOLD = 10_000; // 이 값 이하의 서브태스크는 더 이상 분할할 수 없다

  public ForkJoinSumCalculator(long[] numbers) {
    this(numbers, 0, numbers.length);
  }

  private ForkJoinSumCalculator(long[] numbers, int start, int end) {
    this.numbers = numbers;
    this.start = start;
    this.end = end;
  }

  @Override
  protected Long compute() {
    int length = end - start;
    if (length <= THRESHOLD) {
      return computeSequentially();
    }
    
    ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length/2);
    leftTask.fork(); // ForkJoinPool의 다른 스레드로 새로 생성한 태스크를 비동기로 실행한다

    ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start + length/2, end);
    Long rightResult = rightTask.compute(); // 두 번쨰 서브태스크를 동기 실행한다. 추가로 분할이 일어날 수 있다
    Long leftResult = leftTask.join();

    return leftResult + rightResult;
  }

  private long computeSequentially() {
    long sum = 0;
    for (int i = start; i < end; i ++) {
      sum += numbers[i];
    }
    return sum;
  }
}
```
- 위 메서드는 n까지 자연수의 덧셈 작업을 병렬로 수행하는 방법을 직관적으로 보여준다
- 다음처럼 실행할 수 있다

```java
public static long forkJoinSum(long n) {
  long[] numbers = LongStream.rangeClosed(1, n).toArray();
  ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
  return new ForkJoinPool().invoke(task);
}
```

- 일반적으로 애플리케이션에서는 둘 이상의 `ForkJoinPool`을 사용하지 않는다
  - 정적 필드에 싱글턴으로 저장한다
  - 디폴트 생성자로 `ForkJoinPool`을 만드는 것은 JVM에서 이요할 수 있는 모든 프로세서가 자유롭게 풀에 접근할 수 있음을 의미한다(?)
  - `Runtime.availableProcessors`의 반환값으로 풀에 사용할 스레드 수를 결정한다

**ForJoinSumCalculator 실행**
<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/170608001-c4df8f9d-2efe-463a-8b36-83c36dccfdb8.png">

- 성능을 측정해보면 병렬 스트림보다 나빠진 것을 확인할 수 있는데 이는 전체 스트림을 long[]으로 변환했기 때문이다

### 포크/조인 프레임워크를 제대로 사용하는 방법
- `join` 메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비 될때까지 호출자를 블록시킨다.
  - 따라서 두 서브태스크가 모두 시작된 다음에 `join`을 호출해야 한다

- `RecursiveTask` 내에서는 `ForkJoinPool`의 `invoke` 메서드를 사용하지 말아야 한다
  - `compute`, `fork` 메서드를 직접호출한다
  - 순차 코드에서 병렬 계산을 시작할 때만 `invoke`를 사용한다

- 서브태스크에 `fork` 메서드를 호출해서 `ForkJoinPool`의 일정을 조절할 수 있다
  - 양쪽 작업 모두에 fork를 호출하는 것보다 한 쪽 작업에 compute를 호출하는 것이 효율적이다
  - 한 태스크는 같은 스레드를 재사용할 수 있다

- 포크/조인 프레임워크를 이용하는 병렬 계산은 디버깅하기 어렵다

- 멀티코어에 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 무조건 빠를 것이라는 생각은 버려야 한다
  - 태스크를 여러 독립적인 태스크로 나눌 수 있어야 한다
  - 각 서브태스크의 실행 시간은 새로운 태스크를 포킹하는데 걸리는 시간보다 길어야 한다


### 작업 훔치기
- 실제로는 코어 개수와 관계없이 적절한 크기로 분할된 많은 태스크를 포킹하는 것이 바람직하다
- 이론적으로는 코어 개수만큼 병렬화된 태스크로 작업을 분할하면 모든 cpu 코어에서 태스크를 실행할 것이고 크기가 같은 각각의 태스크는 같은 시간에 종료될 것이라 생각할 수 있다
  - 하지만 실제로는 각각의 서브 태스크 완료시간이 달라질 수 있다
- 포크/조인 프레임워크에서는 작업 훔치기(task stealing)이라는 기법으로 이 문제를 해결한다
  - `ForkJoinPool`의 모든 스레드를 거의 공정하기 분할한다
  - 한 스레드가 작업이 종료된 경우 유휴상태로 있는 것이 아니라 다른 스레드의 작업 큐의 꼬리에서 작업을 훔쳐온다