# 람다를 이용한 도메인 전용 언어
- 프로그래밍 언어는 결국 언어다
  - 사람들이 이해할 수 있도록 작성되어야 한다
  - 의도가 명확히 전달되어야 한다
- 도메인 전용 언어(Domin Specific Language)는 특정 도메인을 대상으로 만들어진 특수한 프로그래밍 언어이다
  - DSL로 애플리케이션의 비즈니스 로직을 표현함으로써 도메인 전문가를 소프트웨어 개발 프로세스에 참여시킬 수 있다

## 도메인 전용 언어
- DSL은 특정 비즈니스 도메인의 문제를 해결하기 위해 만든 언어이다
  - DSL의 동작과 용어는 특정 도메인에 국한되므로 다른 문제는 걱정할 필요가 없어진다
  - 사용자가 특정 도메인의 복잡성을 더 잘 다를 수 있게된다
  - 저수준 구현 세부 사항 메서드는 클래스의 비공개로 만들어서 저수준 구현 세부 내용은 숨길 수 있다
- DSL은 평문 영어가 아니기 때문에 다음 두 가지 필요성을 생각해야 한다
  - 의사 소통의 왕
    - 코드 의도가 명확히 전달되어야 하며, 프로그래머가 아닌 사람도 이해할 수 있어야 한다
  - 한 번 코드를 구현하지만 여러 번 읽는다
    - 동료가 쉽게 이해할 수 있도록 코드를 구현해야 한다

### DSL의 장점과 단점
**장점**
- 간결함: API는 비즈니스 로직을 간편하게 캡슐화하므로 반복을 피할 수 있고 코드를 간결하게 만들 수 있다
- 가독성: 도메인 영역의 용어를 사용하므로 비 도메인 전문가도 코드를 쉽게 이해할 수 있다. 결과적으로 다양한 조직 구성원 간에 코드와 도메인 영역이 공유될 수 있다
- 유지보수: 잘 설계된 DSL로 구현한 코드는 쉽게 유지보수하고 바꿀 수 있다. 유지보수는 비즈니스 관련 코드 즉 가장 빈번히 바뀌는 애플리케이션 부분이 특히 중요하다
- 높은 수준의 추상화: DSL은 도메인과 같은 추상화 수준에서 동작하므로 도메인의 문제와 직접적으로 관련되지 않은 세부 사항을 숨긴다
- 집중: 비즈니스 도메인의 규칙을 표현할 목적으로 설계된 언어이므로 프로그래머가 특정 코드에 집중할 수 있다
- 관심사 분리: 지정된 언어로 비즈니스 로직을 표현함으로 애플리케이션의 인프라구조와 관련된 문제와 독립적으로 비즈니스 관련된 코드에서 집중하기가 용이하다

**단점**
- DSL 설계의 어려움: 간결하게 제한적인 언어에서 도메인 지식을 담는 것이 쉽지 않다
- 개발 비용: 코드에 DSL을 추가하는 작업은 초기 프로젝트에 많은 비용과 시간이 소모되는 작업이다. DSL의 유집수와 변경은 프로젝트에 부담을 주는 요소다
- 추가 우회 계층: DSL은 추가적인 계층으로 도메인 모델을 감싸며 이때 계층을 최대한 작게 만들어 성능 문제를 회피한다
- 새로 배워야 하는 언어: 요즘에는 한 프로젝트에도 여러가지 언어를 사용하는 추세다. DSL을 사용하면 팀에 언어를 하나 더 배워야하는 부담을 지우는 것이다.
- 호스팅 언어 한계: 자바 같이 장황하고 엄격한 문법을 가진 언어를 가지고 DSL을 만든는 것은 쉽지 않다. 자바 8의 람다 표현식은 이러한 문제를 해결할 강력한 도구이다

### JVM에서 이용할 수 있는 다른 DSL 해결책

**내부 DSL**
- 내부 DSL은 기존 호스팅 언어로 개발한 DSL을 의미한다
- 자바는 람다 표현식이 나타나면서 유연성이 떨어지는 문법으로 인한 단점이 완화되었다
- 람다를 적극적으로 사용하면 익명 내부 클래스를 이용해 DSL을 구현하는 것 보다 장황함을 크게 줄여 신호 대비 잡음 비율을 적정 수준으로 유지하는 DSL을 만들 수 있다
- 순수 자바로 DSL을 구현함으로써 얻을 수 있는 장점은 아래와 같다
  - 외부 DSL에 비해 새로운 패턴과 기술을 배워 DSL을 구현하는 노력이 현저히 줄어든다
  - 순수 자바로 DSL을 구현하면 나머지 코드와 함께 DSL을 컴파일할 수 있다
  - 개발 팀이 새로운 언어를 배울 필요가 없다
  - IDE에서 자바를 사용할때와 같이 리팩터링 기능 등 도움을 받을 수 있다
  - 한 개의 언어로 한 개의 도메인 또는 여러 도메인을 대응하지 못해 추가로 DSL을 개발해야 하는 상황에서 자바을 이용한다면 추가 DSL을 쉽게 합칠 수 있다

**다중 DSL**
- 같은 자바 바이트코드를 이용하는 JVM 기반 프로그래밍 언어를 이용함으로 DSL 합침 문제를 해결할 수 있는데, 이를 다중 DSL이라 한다
- 다음과 같은 단점을 가진다
  - 새로운 프로그래밍 언어를 배우거나 팀의 누군가가 이미 해당 기술을 가지고 있어야 한다
  - 두 개 이상의 언어가 혼재하므로 여러 컴파일러로 소스를 빌드하도록 빌드 과정을 개선해야 한다
  - 자바와의 호환성이 완벽하지 않을 떄가 많다

**외부 DSL**
- 자신만의 문법과 구문으로 새 언어를 설계해야 한다
  - 새 언어를 파싱하고, 파서의 결과를 분석하고, 외부 DSL을 실행할 코드를 만들어야 한다
- 장점은 아래와 같다
  - 외부 DSL은 무한한 유연성을 제공한다
  - 자바로 개발된 인프라구조 코드와 외부 DSL로 구현한 비즈니스 코드를 명확하게 분리한다
- 단점은 아래와 같다
  - 인프라구조 코드와 비즈니스 코드를 분리하면서 DSL과 호스트 언어 사이에 인공 계층이 생긴다

## 최산 자바 API의 작은 DSL
- 자바의 새로운 기능의 장점을 적용한 첫 API는 네이티브 자바 API 자신이다
- 자바 8의 Comparator 인터페이스에서 추가된 새로운 메서드와 람다를 통해서 네이티브 자바 API의 재사용성과 메서드 결합도를 높였다

```java
Collections.sort(persons, new Comparator<Person>() {
    public int compare(Person p1, Person p2) {
        return p1.getAge() - p2.getAge();
    }
});
```
- 익명 클래스를 람다로 표현할 수 있다

```java
Collections.sort(persons, (p1, p2) -> p1.getAge() - p2.getAge());
```
- 그리고 자바는 Comparator 객체를 좀 더 가독성 있게 구현할 수 있는 정적 유틸리티 메서드 집합도 제공한다
  - 이들 메서드는 Comparator 인터페이스에 포함되어 있다

```java
Collections.sort(persons, comparing(p -> p.getAge()));
```
- 그리고 `reverse`를 이용해 역순 정렬 가능하고, 메서드 참조로 간단하게 표현할 수 있다

```java
Collections.sort(persons, comparing(Person::getAge).reverse());
```
- `thenComparing`를 이용해 같은 나이의 사람들을 알파벳 순으로 정렬할 수도 있다

```java
Collections.sort(persons, comparing(Person::getAge)
                            .thenComparing(Person::getName));
```

### 스트림 API는 컬렉션을 조작하는 DSL
- Stream 인터페이스는 네이티브 자바 API에 작은 내부 DSL을 적용한 예이다

```java
List<String> errors = new ArrayList<>();
int errorCount = 0;
BufferedReader bufferedReader = new BufferedReader(new FileReader(fileName));
String line = bufferedReader.readLine();
while (errorCount < 40 && line != null) {
    if (line.startsWith("ERROR")) {
        errors.add(line);
        errorCount++;
    }
    line = bufferedReader.readLine();
}
```

- 위 코드는 문제가 분리되지 않아 가독성과 유지보수성 모두 저하되었다
- 같은 의무를 지닌 코드가 여러 행에 분산되어 있다.
  - FileReader가 만들어짐
  - 파일이 종료되었는지 확인하는 while 루프의 두 번째 조건
  - 파일의 다음 행을 읽는 while 루프의 마지막 행
- 첫 40행을 수집하는 코드도 세 부분으로 흩어져있다
  - errorCount 변수를 초기화 하는 코드
  - while 루프의 첫 번째 조건
  - "Error"을 로그에서 발견하면 카운터를 증가시키는 행

- Stream을 사용하면 간결하게 코드를 구현할 수 있다

```java
List<String> errors - Files.lines(Path.get(fileName)) // 파일 끝까지 읽음
                            .filter(line -> line.startsWith("ERROR")) // ERROR로 시작하는 것만
                            .limit(40) // 40행으로 제한
                            .collect(toList());
```

### 데이터를 수집하는 DSL인 Collectors
- Stream 인터페이스는 데이터 리스트를 조작하는 DSL로 간주할 수 있다
- 마찬가지로 Collectors 인터페이스는 데이터 수집을 수행하는 DSL로 간주할 수 있다
- Collectors는 정적 팩토리 메서드를 이용해 필요한 Collector 객체를 만들고 합칠 수 있다
  - Collector 인터페이스를 이용해 스트림의 항목을 수집, 그룹화, 파티션할 수 있다

```java
Map<String, Map<Color, List<Car>>> carsByBrandAndColor = 
    cars.stream()
        .collect(groupingBy(Car::getBrand, 
                            groupingBy(Car::getColor)));
```

- 두 Comparators를 연결하는 방식은 아래와 같다

```java
Comparator<Persom> comparator =
    comparing(Person::getAge).thenComparing(Person::getName);
```

- Collectors API를 이용해 Collectors를 중첩하면 다중 수준의 Collector를 만들 수 있다

```java
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> carGroupingCollector = 
    groupingBy(Car::getBrand, groupingBy(Car::getColor));
```

- 셋 이상의 컴포넌트를 조합할 때는 보통 플루언트 형식이 중첩 형식에 비해 가독성이 좋다
- 중접 형식에서 사실 가장 안쪽의 Collector가 첫 번째로 평가되어야 하지만, 논리적으로는 최종 그룹화에 해당하므로 서로 다른 형식은 이를 어떻게 처리하는 지를 상반적으로 보여준다
- 중접 형식에서는 Collector 생성을 여러 정적 메서드로 중첩함으로 안쪽 그룹화가 처음 평가되고 코드에서는 반대로 가장 나중에 등장하게 된다
  - 그러니까 평가는 가장 빨리되고, 코드에서는 가장 늦게 등장한다?

- GroupingBuilder를 만들면 문제를 더 쉽게 해결할 수 있다
  - GroupingBuilder는 유연한 방식으로 여러 그룹화 작업을 만든다

```java
import static java.util.stream.Collectors.groupingBy;
public class GroupingBuilder<T, D, K> {
    private final Collector<? super T, ?, Map<K, D>> collector;

    private GroupingBuilder(Collector<? super T, ?, Map<K, D>> collector) {
        this.collector = collector;
    }

    public Collector<? super T, ?, Map<K, D>> get() {
        return collector;
    }

    public <J> GroupingBuilder<T, Map<K, D>, J> after(Function<? super T, ? extends J> classifier) {
        return new GroupingBuilder<>(groupingBy(classifier, collector));
    }

    public static <T, D, K> GroupingBuilder<T, List<T>, K> groupOn(Function<? super T, ? extends K> classifier) {
        return new GroupingBuilder<>(groupingBy(classifier));
    }
}
```
- 플루언트 형식 빌더의 문제를 보여주는 코드는 아래와 같다

```java
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>>
    carGroupingCollector = 
        groupOn(Car::getColor).after(Car::getBrand).get();
```

- 중첩된 그룹화 수준에 반대로 그룹화 함수를 구현해야 하므로 유틸리티 사용코드가 직관적이지 않다
- 자바 형식 시스템으로는 이런 순서 문제를 해결할 수 없다

## 자바로 DSL을 만드는 패턴과 기법
- 도메인 모델

```java
@Getter
@Setter
public class Stock {
  private String symbol;
  private String market;
}
```

```java
@Getter
@Setter
public class Trade {
  public enum Type { BUY, SELL }
  private Type type;

  private Stock stock
  private int quantity;
  private double price;

  public double getValue() {
    return quantity * price;
  }
}
```

```java
public class Order {
  private String customer;
  private List<Trade> trades = new ArrayList<>();

  public void addTrade(Trade trade) {
    this.trades.add(trade);
  }

  public String getCustomer() {
    return this.customer;
  }

  public void setCustomer(String customer) {
    this.customer = customer;
  }

  public double getValue() {
    return trades.stream().mapToDouble(Trade::getValue).sum();
  }
}
```
- BigBank 고객이 요청한 두 거래를 포함하는 주문

```java
Order order = new Order();
order.setCustomer("BigBank");

Trade trade1 = new Trade():
trade1.setType(Trade.Type.BUY);

Stock stock1 = new Stock();
stock1.setSymbol("IBM");
stock1.setMarket("NYSE");

trade1.setStock(stock1);
trade1.setPrice(125.00);
trade1.setQuantity(80);
order.addTrade(trade1);

Trade trade2 = new Trade():
trade2.setType(Trade.Type.BUY);

Stock stock2 = new Stock();
stock2.setSymbol("IBM");
stock2.setMarket("NYSE");

trade2.setStock(stock2);
trade2.setPrice(125.00);
trade2.setQuantity(80);
order.addTrade(trade2);
```

### 메서드 체인
```java
Order order = forCustomer("BigBank")
              .buy(80)
              .stock("IBM")
              .on("NYSE")
              .at(125.00)
              .sell(50)
              .stock("GOOGLE")
              .on("NASDAQ")
              .at(375.00)
              .end();
```

- 메서드 체인 기법을 사용하면 기존의 장황하고 보기 어려운 코드를 개선할 수 있다
- 이 기법은 정적 메서드 사용을 최소화하고 메서드 이름이 인수의 이름을 대신하도록 만듦으로 이런 형식의 DSL의 가독성을 개선하는 효과를 더한다
- 메서드 체인 기법은 메서드 체인을 구현해야 한다는 단점을 가진다
- 들여쓰기를 강제하지 않는다는 단점도 가지고 있다

### 중첩된 함수 이용

```java
Order order = order("BigBank",
                    buy(80,
                        stock("IBM", on("NYSE")), at(125.00)),
                    sell(50,
                        stock("GOOGLE", on("NASDAQ")), at(375.00)));
```
- 장점
  - 메서드 체인에 비해 함수의 중첩 방식이 도메인 객체 계층 구조에 그대로 반영된다
    - 예제에서 주문은 한 개 이상의 거래를 포함하며 각 거래는 한 개의 주식을 참조
- 단점
  - 결과 DSL에 더 많은 괄호를 사용해야 한다
  - 인수 목록을 정적 메서드에 넘겨줘야 한다
  - 선택 사항 필드가 있는 경우 이 가능성을 처리할 수 있도록 여러 메서드 오버라이드를 구현해야 한다
  - 인수의 의미가 아니라 위치에 의해 정의 되었다

### 람다 표현식을 이용한 함수 시퀀싱
```java
Order order = order(o -> {
  o.forCustomer("BigBank");
  o.buy(t -> {
    t.quantity(80);
    t.price(125.00);
    t.stock(s -> {
      s.symbol("IBM");
      s.market("NYSE");
    })
  });
  o.sell(t -> {
    t.quantity(50);
    t.price(375.00);
    t.stock(s -> {
      s.symbol("GOOGLE");
      s.market("NASDAQ");
    })
  });
})
```

- 이런 DSL을 만들기 위해서는 람다 표현식을 받아 실행해 도메인 모델을 만들어 내는 여러 빌더를 구현해야 한다
- 장점
  - 메서드 체인 패턴처럼 플루언트 방식으로 거래 주문을 정의한다
  - 중첩 합수 형식처럼 다양한 람다 표현식의 중첩 수준과 비슷하게 도메인 객체 계층 구조를 유지한다
- 단점
  - 많은 설정 코드가 필요하다
  - DSL자체가 자바 8 람다 표현식 문법에 의한 잡음의 영향을 받는다

### 조합하기
```java
Order order = 
  forCustomer("BigBank", 
                buy(t -> t.quantity(80)
                          .stock("IBM")
                          .on("NYSE")
                          .at(125.00),
                sell(t -> t.quantity(50)
                            .stock("GOOGLE")
                            .on("NASDAQ")
                            .at(125.00))));
```

- 장점
  - 지금까지 나온 패턴의 장점을 가짐
- 단점
  - 여러 패턴이 사용되었기 때문에 DSL을 배우는 시간이 오래 걸린다

### DSL에 메서드 참조 사용하기
- 생략

## 실행활의 자바8 DSL
### jOOQ
- SQL을 구현하는 내부적 DSL로 자바에 직접 내정된 형식 안전 언어이다
- 데이터베이스 스키마를 역공한하는 소스코드 생성기 덕분에 자바 컴파일러가 복잡한 SQL 구문의 형식을 확인할 수 있다

```sql
SELECT * FROM BOOK
WHERE BOOK.PUBLISHED_IN = 2016
ORDER BY BOOK.TITLE:
```

```java
create.selectFrom(BOOK)
      .where(BOOK.PUBLISHED_IN.eq(2016))
      .orderBy(BOOK.TITLE);
```

- jOOQ의 또 다른 장점은 스트림 API와 조합해 사용할 수 있다는 것이다

![image](https://user-images.githubusercontent.com/60502370/174195906-9b69a76d-2efb-4d26-9404-abafaf09b011.png)

### 큐컴버
- 행위 주도 개발(BDD)는 테스트 주도 개발의 확장으로 다양한 비즈니스 시나리오를 구조적으로 서술하는 간단한 도메인 전용 스크립팅 언어를 사용한다
- 작성된 명령문을 실행할 수 있는 테스트 케이스로 변환한다
- 결과물은 테스트와 동시에 비즈니스 기능의 수용 기준이 된다
- 도메인 전문가와 프로그래머 사이의 간격을 줄인다

```text
Feature: Buy stock
  Scenario: Buy 10 IBM stocks
    Given the price of a "IBM" stock is 125$
    When I buy 10 "IBM"
    Then the order value should be 1250$
```

- 큐컴버는 전제 조건 정의(Given), 시험하려는 도메인 객체의 실질 호출(When), 테스트 케이스의 결과를 확인하는 어설션(Then) 세 가지 개념을 사용한다

### 스프링 통함
- 유명한 엔터프라이즈 통합 패턴을 지원할 수 있도록 의존성 주입에 기반한 스플이 프로그래밍 모델을 확장한다
- 스프링 통합의 핵심 목표는 복잡한 엔터프라이즈 통합 솔루션을 구현하는 단순한 모델을 제공하고 비동기, 메시지 주도 아키텍처를 쉽게 적용할 수 있게 돕는 것이다
- 스프링 통합은 스프링 기반 애플리케이션 내의 경량의 원격, 메시징, 스케줄링을 지원한다
  
![image](https://user-images.githubusercontent.com/60502370/174196587-7d75371f-e4df-459b-ab5b-9a89a802e211.png)

- 스프링 통합 DSL에서 가장 널리 사용하는 패턴은 메서드 체인이다