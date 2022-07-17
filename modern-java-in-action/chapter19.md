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