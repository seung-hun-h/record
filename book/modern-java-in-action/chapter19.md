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

## 패턴 매칭
- 패턴 매칭은 함수형 프로그래밍을 구분하는 또 하나의 중요한 특징이다
  - 정규표현식과 관련된 패턴 매칭과는 다르다

```java
// 함수형 프로그래밍 패턴 매칭
f(0) = 1
f(n) = n * f(n - 1) // 그렇지 않으면
// 자바 패턴 매칭
if(...) {

} else if (...) {

} else {

}

switch(some) {
  case ...:
  case ...:
  ...
}
```

- 숫자와 바이너리 연산자로 구성된 간단한 수학언어가 있다고 가정
  - 표현식을 단순화하는 메서드를 구현해야 한다
  - 5 + 0 은 5로 단순화 할 수 있다
  - 즉, `new BinOp("+", new Number(5), new Number(0))`은 `new Number(5)`로 단순화 할 수 있다

```java
Expr simplifyExpression(Expr expr) {
  if (expr instanceOf BinOp
      && ((BinOp)expr).opname.equals("+")
      && ((BinOp)expr).right instance Of Number
      && ...
    ) {
      return (BinOp)expr.left;
    }
}
```
- 위 코드는 깔끔하지 않다

### 방문자 디자인 패턴
- 자바에서는 방문자 디자인 패턴으로 자료형을 언랩할 수 있다
  - 특히 특정 데이터 형식을 방문하는 알고리즘을 캡슐화하는 클래스를 따로 만들 수 있다


- 방문자 클래스는 지정된 데이터 형식의 인스턴스를 입력 받는다
- 그리고 인스턴스의 모든 멤버에 접근한다

```java
class BinOp extends Expr {
  ...
  public Expr accept(SimplifyExprVisitor v) {
    return v.visit(this);
  }
}

public class SimplifyExprVisitor {
  ...
  public Expr visit(BinOp e) {
    if ("+".equals(e.opname) && e.right instanceOf Number && ... ) {
      return e.left;
    }
    return e;
  }
}
```

### 패턴 매칭의 힘
- 패턴 매칭으로 좀 더 단순하게 해결 할 수 있다

```s
def simplifyExpression(expr: Expr): Expr = expr match {
  case BinOp("+", e, Number(0)) => e
  case BinOp("*", e, Number(1)) => e
  case BinOp("/", e, Number(1)) => e
  case _ => expr
}
```

## 기타 정보
### 캐싱 또는 기억화
- 트리 형식의 토폴로지를 갖는 네트워크 범위 내에서 존재하는 노드위 수를 계산하는 `computeNumberOfNodes(Range`라는 부작용 없는 메서드가 존재한다고 가정한다
- 이 메서드는 구조체를 재귀적으로 탐색해야하기 때문에 노드 계산 비용이 비싸다
- 참조 투명성이 유지되는 상황이라면 기억화(메모이제이션)를 사용하여 비용을 줄일 수 있다

```java
final Map<Range, Integer> numberOfNodes = new HashMap<>();
Integer computeNumberOfNodesUsingCache(Range range) {
  Integer result = numberOfNodes.get(range);
  if (result != null) {
    return result;
  }

  result = computeNumberOfNodes(range);
  numberOfNodes.put(range, result);
  return result;
}

Integer computeNumberOfNodesUsingCache(Range range) {
  return numberOfNodes.computeIfAbsent(range, this::computeNumberOfNodes);
}
```

- `computeNumberOfNodesUsingCache`는 참조 투명성을 갖는다
- 하지만 numberOfNodes는 공유된 가변 상태이며, HashMap은 동기화 되지 않았으므로, 스레드 안전성이 없는 코드이다
  - ConcurrentHashMap을 사용할 수는 있지만 성능이 크게 저하될 수도 있다
- 함수형 프로그래밍을 사용해서 동시성과 가변 상태가 만나는 상황을 완전히 없앨 수 있다
  - 캐싱 같은 저수준 성능 문제는 해결되지 않는다.
  - 하지만 호출하려는 메서드가 공유된 가변 상태를 포함하지 않음을 미리 알 수 있으므로 동기화 등을 신경 쓸 필요가 없어진다

### 같은 객체를 반환함은 무엇을 의미하는가
- 참조 투명성이란 인수가 같다면 결과도 같아야한다 라는 구칙을 만족함을 의미한다
- 자료구조를 변경하지 않는 상황에서 참조가 다르다는 것은 큰 의미가 없다. 논리적으로 두 객체는 같다고 판단할 수 있다

### 콤비네이터
- 함수형 프로그래밍에서는 두 함수를 인수로 받아 다른 함수를 반환하는 등 고차원 함수를 많이 사용한다
- 함수의 기능을 조합하는 것을 콤비네이터라고 한다
- 자바 8에서는 콤비네이터의 영향을 받았다
  - comparing, andThen, compose 등이 고차원 함수를 제공한다  