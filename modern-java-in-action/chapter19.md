# 함수형 프로그래밍 기법
## 함수는 모든 곳에 존재한다
- 함수형 프로그래밍이란 함수나 메서드가 수학의 함수처럼 부작용 없이 동작함을 의미한다
- 더 폭 넓은 의미의 함수형 프로그래밍이란 함수를 마치 일반값처럼 사용해서 인수로 전달하거나 결과로 반환 받거나 자료구조에 저장할 수 있음을 의미한다
  - 일반값처럼 취급할 수 있는 함수를 일급 함수라고 한다

### 고차원 함수
- 다음 중 하나 이상의 동작을 수행하는 함수를 고차원 함수라고 한다
  - 하나 이상의 함수를 인수로 받음
  - 함수를 결과로 반환

### 커링
- 커링은 x와 y라는 두 인수를 받는 함수 f를 한 개의 인수를 받는 g라는 함수로 대체하는 기법이다
  - g 함수 역시 하나의 인수를 받는 함수를 반환한다
  - 함수 g와 원래 함수 f가 최종적으로 반환하는 값은 같다
  - f(x, y) = (g(x))(y)

```java
// 3개의 인자를 받는 함수
static double converter(double x, double f, double b) {
    return x * f + b;
}

// 커링
static DoubleUnaryOperator curriedConverter(double f, double b) {
    return (double x) -> x * f + b;
}
```

- 커링을 활용하면 로직을 재활용할 수 있고, 다양한 변환 요소로 다양한 함수를 만들 수 있다

## 영속 자료구조
- 영속 자료구조는 함수형 프로그램에서 사용하는 자료구조이다
  - 데이터베어스에서 사용하는 영속과는 다른 의미이다
- 함수형 메서드에서는 전역 자료구조나 인수로 전달된 구조를 갱신할 수 없다
  - 자료구조를 바꾼다면 같은 메서드를 두 번 호출했을 때 결과가 달라지면서 참조 투명성에 위배되고 인수를 결과로 단순하게 매핑할 수 있는 능력이 상실된다

### 파괴적인 갱신과 함수형
- A에서 B까지 기차여행을 의미하는 **가변** TrainJourney 클래스가 있다고 가정한다

```java
class TrainJourney {
    public int price;
    public TrainJourney onward;
    public TrainJourney(int p, TrainJourney t) {
        this.price = p;
        this.onward = t;
    }
}
```
- X에서 Y로 이어지는 여행과 Y에서 Z까지의 여행을 나타내는 별도의 객체가 있다고 가정한다
- 두 개의 객체를 연결해서 하나의 여행을 만들 수 있다

```java
static TrainJourney link(TrainJourney a, TrainJourney b) {
    if (a == null) return b;
    TrainJourney t = a;
    while (t.onward != null) {
        t = t.onward;
    }

    t.onward = b;
    return a;
}
```
- 위 코드는 버그가 존재한다
  - X에서 Y까지의 여행 first 객체가 있고, Y에서 Z까지의 여행 second가 있다고 가정한다
  - `TrainJourney(first, second)`를 호출한 결과 first는 X에서 Z까지의 여행으로 변경된다
  - 서울 - 대구, 대구 - 부산 여행에서 서울 - 부산으로 변경되는 것이다

- 함수형 프로그래밍에서는 위 같은 버그를 해결하기 위해서 기존 자료구조를 갱신하지 않고 새로운 자료구조를 만든다
- 이는 표준 객체지향 프로그래밍 관점에서도 좋은 기법이다

```java
static TrainJourney append(TrainJourney a, TrainJourney b) {
    return a == null ? b : new TrainJourney(a.price, append(a.onward, b));
}
```

- 위 함수는 명확하게 함수형이며 기존 자료구조를 변경하지 않는다.

## 스트림과 게으른 평가
- 스트림은 단 한 번만 소비할 수 있어서 재귀적으로 정의할 수 없다.

### 자기 정의 스트림
- 소수 스트림 생성 코드

```java
public static Stream<Integer> primes(int n) {
  return Stream.iterate(2, i -> i + 1)
                .filter(MyMathUtils::isPrime)
                .limit(n);
}

//MyMathUtils
public static boolean isPrime(int candidate) {
  int candidateRoot = (int) Math.sqrt((double) candidate);
  return IntStream.rangeClosed(2, candidateRoot)
                  .noneMatch(i -> candidate % i == 0);
}
```
- 위 코드는 candidate로 정확히 나누어 떨어지는 지 매번 모든 수를 반복 확인하므로, 좋은 코드는 아니다
- 이론적으로 재귀 스트림으로 소수 스트림을 생성할 수 있다
  1. 소수를 선택할 숫자 스트림이 필요하다
  2. 스트림에서 첫 번째 수를 가져온다. 이 숫자는 소수다
  3. 스트림의 꼬리에서 가져온 수로 나누어떨어지는 모둔 수를 걸러 제외시킨다
  4. 남은 숫자만 포함하는 새로운 스트림에서 소수를 찾는다. 다시 1번부터 다시 이 과정을 반복하게된다

```java
// 1. 스트림 숫자 얻기
static IntStream numbers() {
  return IntStream.iterate(2, n -> n + 1);
}

// 2. 머리 획득
static int head(IntStream numbers) {
  return numbers.findFirst().getAsInt();
}

// 3. 꼬리 필터링
static IntStream tail(IntStream numbers) {
  return numbers.skip(1);
}

// 4. 재귀적으로 소수 스트림 생성
static IntStream primes(IntStream numbers) {
  int head = head(numbers);
  return IntStream.concat(
    IntStream.of(head),
    primes(tail(numbers).filter(n -> n % head != 0))
  );
}
```
- 4 단계 코드를 실행하면 예외가 발생한다
  - "java.lang.IllegalStateException: stream has already been operated upon or closed"
  - 스트림의 머리와 꼬리로 분리하는 두 개의 최종 연산 findFirst, skip을 사용했기 때문이다.
  - 최종 연산을 스트림에 호출하면 스트림은 완전히 소비된다

**게으른 평가**
- 그리고 IntStream.concat 내부에서 primes를 재귀적을 호출하여 무한 재귀에 빠진다
- 자바 8의 스트림의 재귀적 정의를 허용하지 않는 규칙 덕분에 데이터베이스 같은 질의를 표현하고 병렬화할 수 있는 능력을 얻을 수 있다
- concat의 두 번째 인수인 primes를 게으르게 평가하여 문제를 해결할 수 있다
  - 소수를 처리할 필요가 있을 때만 스트림을 실제로 평가한다

### 게으른 리스트 만들기
- 자바 8의 스트림은 게으르다
  - 스트림에 일련의 연산을 적용하면 연산이 수행되지 않고 일단 저장된다
  - 스트림에 최종 연산을 적용해서 실제 계산을 해야 하는 상황에서만 실제 연산이 이루어진다
  - 게으른 특성 덕분에 각 연산별로 스트림을 탐색할 필요 없이 한 번에 여러 연산을 처리할 수 있다

- 게으른 리스트는 스트림과 비슷한 개념으로 구성되며, 고차원 함수도 지원한다

```java
class LazyList<T> implements MyList<T> {
  final T head;
  final Supplier<MyList<T>> tail;
  public LazyList(T head, Supplier<MyList<T>> tail) {
    this.head = head;
    this.tail = tail;
  }

  public T head() {
    return head;
  }

  public MyList<T> tail() {
    return tail.get();
  }

  public boolean isEmpty() {
    return false;
  }
}
```

- 소수 생성

```java
public static MyList<Integer> primes(MyList<Integer> numbers) {
  return new LazyList<>(
    numbers.head(),
    () -> primes(
      numbers.tail()
              .filter(n -> n % numbers.head() != 0)
    )
  );
}
```
- 위 코드에서 MyList는 filter 메서드를 정의하지 않으므로 컴파일 에러가 발생한다

**게으른 필터 구현**
```java
public MyList<T> filter(Predicate<T> p) {
  return isEmpty() ?
          this:
          p.test(head()) ?
            new LazyList<>(head(), () -> tail.filter(p)) :
            tail().filter(p);
}
```