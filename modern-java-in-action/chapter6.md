## 목차

# 스트림으로 데이터 수집
```java
//예시 코드
Map<Currency, List<Transaction>> transactionsByCurrencies = 
    transactions.stream.collect(groupingBy(Transaction::getCurrency));

```

## 컬렉터란 무엇인가?
- Collector 인터페이스 구현은 스트림의 요소를 어떤 식으로 도출할 지 지정한다
- `toList()`는 각 요소를 리스트로 만든다
- `groupingBy()`는 각 키 버킷 그리고 각 키 버킷에 대응하는 요소 리스트를 값으로 포함하는 맵을 만든다

```java
// java.util.stream.Collectors
public static <T>
    Collector<T, ?, List<T>> toList() {
        return new CollectorImpl<>(ArrayList::new, List::add,
                                   (left, right) -> { left.addAll(right); return left; },
                                   CH_ID);
}

public static <T, K, D, A, M extends Map<K, D>>
    Collector<T, ?, M> groupingBy(Function<? super T, ? extends K> classifier,
                                  Supplier<M> mapFactory,
                                  Collector<? super T, A, D> downstream) {
            ....
            return new CollectorImpl<>(...);
}

```
- 명령형 코드에서는 다수준 그룹화를 진행할 때 다중 루프문과 조건문을 추가해 가독성과 유지보수성이 크게 떨어진다
- 함수형 프로그래밍에서는 필요한 컬렉터를 쉽게 추가할 수 있다

### 고급 리듀싱 기능을 수행하는 컬렉터
- 훌륭하게 설계된 함수형 API의 또 다른 장점으로는 높은 수준의 조합성과 재사용성이 있다
- `collect`로 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의할 수 있다는 점이 컬렉터의 최대 강점이다
- 스트림에 `collect`를 호출하면 스트림의 요소에 (컬렉터로 파라미터화 된) 리듀싱 연산이 수행된다
- 보통 함수를 요소로 변환할 때는 컬렉터를 적용하며 최종 결과를 저장하는 자료구조에 값을 누적한다

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/169164148-2e51ca09-9dfd-442a-b6e6-cb3ad890e9b4.png">

- Collector 인터페이스의 메서드를 어떻게 구현하느냐에 따라 스트림에 어떤 리듀싱 연산을 수행할 지 결정된다
- Collectors 유틸리티 클래스는 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 정적 팩토리 메서드를 제공한다

### 미리 정의된 컬렉터
- Collectors에서 제공하는 메서드의 기능은 크게 세 가지로 나뉜다
  - 스트림 요소를 하나의 값으로 리듀스하고 요약
  - 요소 그룹화
  - 요소 분할

## 리듀싱과 요약
- 다양한 계산을 수행할 때 이들 컬렉터를 유용하게 사용할 수 있다

### 스트림값에서 최대값과 최소값 검색
- `Collectors.maxBy`, `Collectors.minBy` 두 개의 메서드를 사용할 수 있다
- 두 컬렉터는 스트림의 요소를 비교하는 데 사용할 Comparator를 인수로 전달받는다

```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);

Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(dishCaloriesComparator));
```
- `Optional`은 빈 스트림을 위함이다
- 스트림에 있는 객체의 숫자 필드의 합계나 평균 등일 반환하는 연산에도 리듀싱 기능이 자주 사용된다. 이러한 연산을 요약이라 한다

### 요약 연산
- Collectors 클래스는 `Collectors.summingInt`라는 특별한 요약 팩토리 메서드를 제공한다
- `summingInt`는 객체를 int로 매핑하는 함수를 인수로 받는다
  - 그리고 `collect` 메서드로 전달되면 요약작업을 수행한다

```java
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

- `summingDouble`, `summingLong` 도 존재한다
- `averagingInt`, `averaginLong`, `averagingDouble` 을 사용해 평균을 구할 수도 있다
- 종종 이들 중 두개 이상의 연산을 한 번에 수행할 수도 있는데 이때 `summarizingInt`가 반환하는 컬렉터를 사용할 수 있다

```java
IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));

IntSummaryStatistics(count=9, sum=4300, min=120, average=477.7777778, max=800)
```

### 문자열 연결
- `Collectors.joining`은 각 객채에 `toString`을 호출해서 추출한 모든 문자열을 하나의 문자열로 연결해서 반환한다

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining());
```

- `joining`은 내부적으로 StringBuilder를 이용해서 문자열을 하나로 만든다
- 연결된 두 요소 사이에 구분자도 넣어줄 수 있다

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
```

### 범용 리듀싱 요약 연산
- 지금까지 살펴본 모든 컬렉터는 `reducing` 팩토리 메서드로도 정의할 수 있다
```java
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i, j) -> i + j));
```
- 첫 번째 인수는 리듀싱 연산의 시작 값
- 두 번째 인수는 요리를 칼로리의 수로 변환
- 세 번째 인수는 같은 종류의 두 항목을 하나의 값으로 더하는 `BinaryOperator`다

- 한 개의 인수를 사용하는 방법도 있다

```java
Optional<Dish> mostCalorieDish = menu.stream().collect(reducing((d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
```

- 한 개의 인수를 갖는 reducing 메서드는 세 개의 인수를 갖는 reducing 메서드의 첫 번째 요소를 인수로 받는다
- 그리고 두 번째 인수를 항등 함수로 받는 상황에 해당한다
- 한 개의 인수를 갖는 reducing은 시작값이 없으므로 Optional을 반환한다

```java
public static <T> Collector<T, ?, Optional<T>>
    reducing(BinaryOperator<T> op) {
        class OptionalBox implements Consumer<T> {
            T value = null;
            boolean present = false;

            @Override
            public void accept(T t) {
                if (present) {
                    value = op.apply(value, t);
                }
                else {
                    value = t;
                    present = true;
                }
            }
        }

        return new CollectorImpl<T, OptionalBox, Optional<T>>(
                OptionalBox::new, OptionalBox::accept,
                (a, b) -> { if (b.present) a.accept(b.value); return a; },
                a -> Optional.ofNullable(a.value), CH_NOID);
}
```
```java
public static <T> Collector<T, ?, T>
    reducing(T identity, BinaryOperator<T> op) {
        return new CollectorImpl<>(
                boxSupplier(identity),
                (a, t) -> { a[0] = op.apply(a[0], t); },
                (a, b) -> { a[0] = op.apply(a[0], b[0]); return a; },
                a -> a[0],
                CH_NOID);
}
```
```java
public static <T, U>
    Collector<T, ?, U> reducing(U identity,
                                Function<? super T, ? extends U> mapper,
                                BinaryOperator<U> op) {
        return new CollectorImpl<>(
                boxSupplier(identity),
                (a, t) -> { a[0] = op.apply(a[0], mapper.apply(t)); },
                (a, b) -> { a[0] = op.apply(a[0], b[0]); return a; },
                a -> a[0], CH_NOID);
}

```

### collect와 reduce
- collect 메서드는 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계됐다
- reduce는 두 값을 하나로 도출하는 불변형 연산이다

```java
Stream<Integer> stream = Arrays.asList(1, 2, 3, 4, 5, 6).stream();
List<Integer> numbers = stream.reduce(new ArrayList<>()
                    , (List<Integer> l, Integer e) -> {
                        l.add(e);
                        return l;
                    }
                    , (List<Integer> l1, List<Integer> l2) -> {
                        l1.addAll(l2);
                        return l1;
                    })
```

- 따라서 위 코드는 reduce를 의미론적으로 잘못사용한 형태이다
  - 누적자로 사용된 리스트를 변환하기 떄문이다
- 의미론적으로 reduce 메서드를 잘못 사용하면 여러 실용성 문제도 발생할 수 있다
  - 여러 스레드가 동시에 같은 데이터 구조체를 고치려면 리스트 자체가 망가져 리듀싱 연산을 병렬로 수행할 수 없다

**컬렉션 프레임 워크의 유연성: 같은 연산도 다양한 방식으로 수행할 수 있다**<br/>
- Integer 클래스의 sum 메서드를 사용하면 코드를 단순화 할 수 있다

```java
// 다양한 구현 방법
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, Integer::sum));

int totalCalories = menu.stream().map(Dish::getCalories).reduce(Integer::sum).get() // Optional

int totalCalories = menu.stream().mapToInt(Dish::getCalories).sum();
```
- 두 번째 경우 Optional이 반환됐는데, 이 경우에는 orElse나 orElseGet 등을 사용하는 것이 더 좋다

- counting 컬렉터도 reducing 팩토리 메서드를 사용해서 구현할 수 있다

```java
public static <T> Collector(T, ?, Long) counting() {
    return reducing(0L, e -> 1L, Long::sum);
}
```

**자신의 상황에 맞는 최적의 해법 선택**<br/>
- 함수형 프로그래밍에서는 하나의 연산을 다양한 방법으로 해결할 수 있다
- 스트림 인터페이스에서 직접 제공하는 메서드를 이용하는 것에 비해 컬렉터를 이용하는 코드가 더 복잡하다
  - 대신 높은 수준의 추상화와 일반화를 통해 재사용성과 커스터마이징 가능성을 제공한다
- 문제를 해결할 수 있는 다양한 해결 방법을 확인한 다음 가장 일반적으로 문제에 특화된 해결책을 고르는 것이 바람직하다
  - 가독성과 성능의 두 마리 토끼를 잡을 수 있다

## 그룹화
- 자바 8의 함수형을 이용하면 가독성있는 한 줄의 코드로 그룹화를 구현할 수 있다
  - 명령형으로 데이터를 그룹화하기 위해서는 많은 코드가 필요하다
- 팩토리 메서드 `Collectors.groupingBy` 예제

```java
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));
```

- `Dish::getType`을 기준으로 스트림이 그룹화되어 이를 **분류 함수(Classification Function)**이라 한다
- 그룹화 연산의 결과로 그룹화 함수가 반환하는 키와 그 키에 대응하는 스트림의 모든 항목 리스트를 값으로 갖는 맵이 반환된다

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/169411830-72ee69b4-7fc6-4690-966b-d66af22c7a1c.png">

- 분류 기준이 더 복잡한 경우에는 람다 표현식으로 직접 로직을 작성할 수 있다

```java
public enum CaloricLevel {DIET, NORMAL, FAT}

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(
    groupingBy(dish -> {
        if (dish.getCalories() <= 400) return CaloricLevel.DIET;
        if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
        return CaloricLevel.FAT;
    });
);
```

### 그룹화된 요소 조작
- 500 칼로리가 넘는 요리만 필터하는 경우
  - `filter`와 프레티케이터를 사용할 수 있다
```java
Map<Dish.Type, List<Dish>> caloricDishesByType = menu.stream()
                        .filter(dish -> dish.getCalories() > 500)
                        .collect(groupingBy(Dish::getType));
// {OTHER=[french fries, pizza], MEAT={pork, beef}}
```
- 위 코드에서는 프레디케이트를 만족하지 못하는 모든 Dish는 그룹화되지 않는다
  - 따라서 FISH와 같은 타입들이 그룹화되지 않을 수 있다
  - 즉, 결과 맵에서 FISH 같은 키 자체가 사라진다
  - FISH에 대한 프레디케이트를 만족하는 요리가 없는 경우 빈 리스트를 반환하는 것이 더 적절하다면 위 코드는 잘못됐다
- `Collectors` 클래스는 일반적인 분류 함수에 **`Collector` 형식의 두 번째 인수**를 갖도록 `groupingBy` 팩토리 메서드를 오버로드해 이 문제를 해결한다
- 두 번째 Collector 안으로 필터 프레디케이트를 이동하여 문제를 해결한다

```java
Map<Dish.Type, List<Dish>> caloricDishesByType = 
                                        menu.stream()
                                            .collect(groupingBy(Dish::getType,
                                                    filtering(dish -> dish.getCalories() > 500, toList())));

// {OTHER=[french fries, pizza], MEAT={pork, beef}, FISH=[]}
```

- 그룹화된 항목을 조작하는 기능 중 맵핑함수도 있다
- `filtering` 컬렉터와 같은 이유로 Collectors 클래스는 `mapping` 메서드를 제공한다

```java
Map<Dish.Type, List<Dish>> caloricDishesByType = 
                                        menu.stream()
                                            .collect(groupingBy(Dish::getType,
                                                    mapping(Dish::getName, toList())));
```

- 각 요리에 여러 태그가 있고, Dish.Type 마다 태그를 추출하는 경우 `flatMapping`을 사용할 수 있다

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/169414373-ee3d360b-b06e-4901-a79d-58c0735a353e.png">

```java
Map<Dish.Type, Set<String>> dishNamesByType = 
    menu.stream()
        .collect(groupingBy(Dish::getType,
                flatMapping(dish -> dishTags.get(dish.getName()).stream(), toSet())));

/**
{MEAT=[salty, greasy, roasted, fried, crisp], FISH=[roasted, tasty, fresh, delicious], OTHER=[salty, greasy, natural, light, tasty, fresh, fried]}
*/
```

### 다수준 그룹화
- `groupingBy`의 두 번째 인수에 `groupingBy`를 사용하면 두 수준으로 스트림 항목을 그룹화 할 수 있다

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = 
    menu.stream()
        .collect(
            groupingBy(Dish::getType,
                groupingBy(dish -> {
                    if (dish.getCalories() <= 400)
                        return CaloricLevel.DIET;
                    if (dish.getCalories() < 700)
                        return CaloricLevel.NORMAL;
                    return CaloricLevel.FAT;
                }))
        );

/**
{MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]}, ...}
*/
```

- 다수준 그룹화 연산은 다양한 수준으로 확장할 수 있다.
  - n 수준의 그룹화 결과는 n 수준 트리 구조로 표현되는 n 수준 맵이 된다.

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/169416666-0b98459d-64f3-4d6a-9f9c-6fd33e758afa.png">

- `groupingBy`의 연산을 버킷 개념으로 생각하면 쉽다

### 서브그룹으로 데이터 수집
- 첫 번째 `groupingBy`로 넘겨주는 컬렉터의 형식에 대한 제한은 없다
- `counting` 컬렉터를 넘겨 메뉴에서 요리 수를 종류별로 계산할 수도 있다

```java
Map<Dish.Type, Long> typesCount = menu.stream().collect(
                            groupingBy(Dish::getType, counting()));

// {MEAT=3, FISH=2, OTHER=4}
```

- 분류 함수 한 개의 인수를 갖는 `groupingBy(f)`는 사실 `groupingBy(f, toList())`의 축약형이다
- 메뉴에서 가장 높은 칼로리를 가진 요리를 찾는 코드

```java
Map<Dish.Type, Optional<Dish>> mostCaloricByType = 
    menu.stream()
        .collect(groupingBy(Dish::getType,
                            maxBy(comparingInt(Dish::getCalories))));

// {FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[pork]}
```

> 맵의 형식이 Optional이 돼었다<br/>
> 하지만 처음부터 존재하지 않는 요리의 키는 맵에 추가되지 않으므로 `Optional.empty()`를 값으로 갖는 요리는 존재하지 않는다<br/>
> `groupingBy` 컬렉터는 스트림의 첫 번째 요소를 찾은 이후에야 그룹화 맵에 새로운 키를 추가한다 리듀싱 컬렉터가 반환하는 형식을 사용하는 상황이므로 굳이 `Optional` 래퍼를 사용할 필요가 없다

**컬렉터 결과를 다른 형식에 적용하기**<br/>
- 마지막 그룹화 연산에서 맵의 모든 값을 Optional로 감쌀 필요가 없으므로 Optional을 제거할 수 있다
  - `Collectors.collectingAndThen`으로 컬렉터가 반환된 결과를 다른 형식으로 활용할 수 있다

```java
Map<Dish.Type, Optional<Dish>> mostCaloricByType = 
    menu.stream()
        .collect(groupingBy(Dish::getType,
                    collectingAndThen(
                            maxBy(comparingInt(Dish::getCalories)),
                            Optional::get)));
```
- `collectingAndThen`은 적용할 컬렉터와 변환 함수를 인수로 받아 다른 컬렉터를 반환한다

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/169418689-e7595ba5-20f7-4191-af7f-35f7a52e2053.png">

**groupingBy와 함께 사용하는 다른 컬렉터 예제**<br/>
- 같은 그룹으로 분류된 모든 요소에 리듀싱 작업을 수행할 때는 팩토리 메서드 `groupingBy`에 두 번째 인수로 전달한 컬렉터를 사용한다

```java
// 각 그룹으로 분류된 요리의 칼로리 합
Map<Dish.Type, Optional<Dish>> totalCaloriesByType = 
    menu.stream()
        .collect(groupingBy(Dish::getType,
                    summingInt(Dish::getCalories)));
```

- `mapping`은 입력 요소를 누적하기 전에 매핑 함수를 적용해서 다양한 형식의 객체를 주어진 형식의 컬렉터에 맞게 변환하는 역할을 한다

```java
Map<Dish.Type, Set<CaloricLevel>> caloricLevelByType = 
    menu.stream()
        .collect(
            groupingBy(Dish::getType,
                mapping(dish -> {
                    if (dish.getCalories() <= 400)
                        return CaloricLevel.DIET;
                    if (dish.getCalories() < 700)
                        return CaloricLevel.NORMAL;
                    return CaloricLevel.FAT;
                }, toSet()))
        );
```

- `toCollection`을 사용하면 원하는 방식으로 결과를 제어할 수도 있다
  - ex: `toCollection(HashSet::new)`

## 분할
- **분할 함수(Partitioning Function)**이라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능이다
  - 불리언을 반환하므로 키는 Boolean이다
  - 그룹화 맵은 최대 두 개의 그룹(true or false)으로 분류된다

```java
// 채식과 채식이 아닌 요리
Map<Boolean, List<Dish>> partitionedMenu = 
    menu.stream()
        .collect(partitioningBy(Dish::isVegetarian));
/**
{false=[port, beef, chicken, prawns, salmon],
true=[french fries, rice, season fruit, pizza]}
*/
```

### 분할의 장점
- 분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다는 것이 분할의 장점이다
  - `filter` 사용하면 프레디케이트 결과가 false인 것들은 유지하지 않는 것에 비해 `partitioningBy`는 유지 한다는 것에 이점을 가진다는 의미인 것 같다
- `partitioningBy`의 두 번쨰 인수로 컬렉터를 전달할 수도 있다

```java
// 채식과 채식이 아닌 요리를 요리 종류 별로 그룹화
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType = 
    menu.stream()
        .collect(
            partitioningBy(Dish::isVegetarian,
                            groupingBy(Dish::getType));
        )
/**
{false={MEAT=[port, beef, chicken], FISH=[prawns, salmon]},
true={OTHER=[french fries, rice, season fruit, pizza]}}
*/
```


```java
// 채식과 채식이 아닌 요리의 가장 칼로리가 높은 요리 종류
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType = 
    menu.stream()
        .collect(
            partitioningBy(Dish::isVegetarian,
                            collectingAndThen(maxBy(Dish::getCalories)),
                            Optional::get);
        )
// {false=pork, true=pizza}
```

### 숫자를 소수와 비소수로 분할하기
```java
public boolean isPrime(int candidate) {
    int candidateRoot = (int) Math.sqrt(candidate);
    return IntStream.rangeClosed(2, candidateRoot)
                    .nonMatch(i -> candidate % i == 0);
}

public Map<Boolean, List<Integer>> partitionPrimes(int n) {
    return IntStream.rangeClosed(2, n)
                    .boxed()
                    .collect(
                        partitioningBy(candidate -> isPrime(candidate)));
}
```

## Collector 인터페이스
- Collector 인터페이스는 리듀싱 연산을 어떻게 구현할지 제공하는 메서드 집합으로 구성된다

```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Characteristics> characteristics();
}
```
- T는 수집될 스트림 항목의 제네릭 형식이다
- A는 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체의 형식이다
- R은 수집 연산 결과 객체의 형식이다

```java
// Stream<T>의 모든 요소를 List<T>로 하는 ToListCollector<T>
public class ToListCollector<T> implements Collector<T, List<T>, List<T>>
```

### Collector 인터페이스의 메서드 살펴보기

**supplier: 새로운 결과 컨테이너 만들기**
- `supplier` 메서드는 빈 결과로 이루어진 Supplier를 반환해야 한다
  - 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수이다

```java
// ToListCollector supplier
// Lambda
public Supplier<List<T>> supplier() {
    return () -> new ArrayList<>();
}
// Method reference
public Supplier<List<T>> supplier() {
    return ArrayList::new;
}
```

**accumulator: 결과 컨테이너에 요소 추가하기**
- 리듀싱 연산을 수행하는 함수를 반환한다
  - 스트림의 n번째 요소를 탐색할 때, 누적자와 n번쨰 요소를 함수에 적용한다
- 반환값은 void
  - 요소를 탐색하며 적용하는 함수에 의해 누적자 내부상태가 바뀌므로 누적자가 어떤 값일지 단정할 수 없다

```java
// Lambda
public BiCunsumer<List<T>, T> accumulator() {
    return (list, item) -> list.add(item);
}

// Method reference
public BiCunsumer<List<T>, T> accumulator() {
    return List::add;
}
```

**finisher: 최종 변환값을 결과 컨테이너로 적용하기**
- 스트림 탐색을 끝내고 누적자 객체를 최정 결과로 반환하면서 누적 과정을 끝낼 때 호출할 함수를 반환한다
  - 누적자 객체가 이미 최정 결과인 경우 항등 함수를 반환한다

```java
public Function<List<T>, List<T>> finisher() {
    return Function.identity();
}
```

- 실제로는 `collect`가 동작하기 전에 다른 중간 연산과 파이프라인을 구성할 수 있게 해주는 게으른 특성 그리고 병렬 실행 등도 고려해야 한다

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/169679039-5c5a1bf3-e07a-407e-87bb-08775fb7fad1.png">

**combiner: 두 결과 컨테이너 병합**
- 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할 지 정의한다

```java
public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> {
        list1.addAll(list2);
        return list1;
    }
}
```

- 네 번째 메서드(`combiner`?)를 이용하면 스트림의 리듀싱을 병렬로 수행할 수 있다
  - 스트림의 리듀싱을 병렬로 수행할 때 자바7의 포크/조인 프레임워크와 `Spliterator`를 사용한다

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/169679225-04fe970a-be90-4094-8b49-50776910e3e8.png"> 

1. 스트림을 분할해야 하는지 정의하는 조건이 거짓으로 바뀌기 전까지 원래 스트림을 재귀적으로 분할한다
  - 분산된 작업의 크기가 너무 작아지면 병렬 수행의 속도는 순차 수행의 속도보다 느려진다
  - 일반적으로 프로세싱 코어의 개수를 초과하는 병렬 작업은 효율적이지 않다
2. 모든 서브스트림의 각 요소에 리듀싱 연산을 순차적으로 적용해서 서브스트림을 병렬로 처리할 수 있다
3. 컬렉터의 `combiner` 메서드가 반환하는 함수로 모든 부분결과를 쌍으로 합친다
   - 분할된 모든 서브스트림의 결과를 합치면서 연산이 종료된다

**Characteristics**
- 컬렉터의 연산을 정의하는 `Characteristics` 형식의 불변 집합을 반환한다
- 스트림을 병렬로 리듀스할 것인지 그리고 병렬로 리듀스한다면 어떤 최적화를 선택해야 할지 힌트를 제공한다
- `Characteristics`는 다음 세 항목을 포함하는 열거형이다
  - `UNORDERED`: 리듀싱의 결과가 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않는다
  - `CONCURRENT`: 다중 스레드에서 `accumulator` 함수를 동시에 호출할 수 있으며 이 컬렉터는 스트림의 병렬 리듀싱을 수행할 수 있다. 컬렉터의 플래그에 `UNORDERED`를 함께 설정하지 않았다면 데이터 소스가 정렬되어 있지 않은 상황에서만 병렬 리듀싱을 수행할 수 있다
  - `IDENTITY_FINISH`: `finisher` 메서드가 반환하는 함수는 단순히 identity를 적용할 뿐이므로 이를 생략할 수 있다. 따라서 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용할 수 있다. 또한 누적자 A의 결과 R로 안전하게 형변환 할 수 있다 

- `ToListCollector`는 `UNORDERED`, `CONCURRENT`, `IDENTITY_FINISH`이다
  - 요소의 순서가 무의미한 데이터 소스여야 병렬로 수행할 수 있다
  - 누적 순서가 필요한 것 아닌가?
    - 누적 순서가 달라도 되면 스트림 소스인 List의 순서가 달라질 것 같은데
  
```java
public class ToLIstCollector<T> implements Collector<T, List<T>, List<T>> {
	@Override
	public Supplier<List<T>> supplier() {
		return ArrayList::new;
	}

	@Override
	public BiConsumer<List<T>, T> accumulator() {
		return List::add;
	}

	@Override
	public BinaryOperator<List<T>> combiner() {
		return (list1, list2) -> {
			list1.addAll(list2);
			return list1;
		};
	}

	@Override
	public Function<List<T>, List<T>> finisher() {
		return Function.identity();
	}

	@Override
	public Set<Characteristics> characteristics() {
		return Collections.unmodifiableSet(EnumSet.of(
            // Characteristics.UNORDERED,
			Characteristics.CONCURRENT,
			Characteristics.IDENTITY_FINISH
		));
	}
}
```

```java
// given
int[] numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// when
List<Integer> result = Arrays.stream(numbers).boxed()
    .parallel()
    .filter(number -> number % 2 == 0)
    .collect(new ToLIstCollector<>());
// then
System.out.println("result = " + result);
```

- `Characteristics.UNORDERED`일 때 
  - 소스의 순서가 보장되지 않음

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/169680225-1f5aeccf-df72-4797-a37e-50929d0a2fdf.png">

- `Characteristics.UNORDERED` 아닐 때 
  - 소스의 순서가 보장됨

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/169680267-f6bdc465-afea-40eb-bcec-f2fbf60f8174.png">


**컬렉터 구현을 만들지 않고도 커스텀 수집 수행하기**
- `IDENTITY_FINISH` 수집 연산에서는 Collector 인터페이스를 완전히 새로 구현하지 않고도 같은 결과를 얻을 수 있다
  - 왜?: 일단 `java.util.stream.Stream<T>`의 `collect`가 그렇게 정의 되어있음
  - `IDENTITY_FINISH`가 아닌 경우는 마지막 반환할 타입으로 값을 변환하는 로직이 구현되어야 한다.
    - 이럴거면 Collector를 구현해라?

```java
<R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner); // Collector의 combiner가 BinaryOperator인 것과 비교됨

<R, A> R collect(Collector<? super T, A, R> collector);
```

```java
List<Dish> dishes = menuStream.collect(
    ArrayList::new, // supplier
    List::add, // accumulator
    List::addAll // combiner
)
```
- 위 코드는 간결하지만 가독성이 떨어진다
  - 적절한 클래스로 커스텀 컬렉터를 구현하는 편이 중복을 피하고 재사용성을 높이는데 도움이 된다
- `collect` 메서드로는 `Characteristics`를 전달할 수 없다
  - 즉, `IDENTITY_FINISH`와 `CONCURRENT`지만 `UNORDERED`는 아닌 컬렉터로만 동작한다
  - 왜?: `IDENTITY_FINISH`면 finisher를 생략할 수 있다. `CONCURRENT`면 다중 스레드에서 `accumulator`를 호출할 수 있다.
  - 왜 `UNORDERED`가 아니어야 하는가?
    - > The combiner function must fold the elements from the second result container into the first result container.
    - https://stackoverflow.com/questions/30526324/where-is-defined-the-combination-order-of-the-combiner-of-collectsupplier-accu
    - Collector의 combiner는 BinaryOperator이므로 새로운 결과를 반환할 수 있는 반면, collect의 combiner는 BiCosumer이므로 어떤 결과를 반환하지 않는다.
      - 따라서 새로운 결과를 반환함으로써 조작을 가할 수 없다. 단지 연산을 적용만할 수 있다

## 커스텀 컬렉터를 구현해서 성능 개선하기
- 자연수를 소수와 비소수로 나누기

```java
public Map<Boolean, List<Integer>> partitionPrimes(int n) {
    return IntStream.rangeClosed(2, n).boxed()
                    .collect(partitioningBy(candidate -> isPrime(candidate)));
}

public boolean isPrime(int candidate) {
    int candidateRoot = (int) Math.sqrt(candidate);
    return IntStream.rangeClosed(2, candidateRoot)
                    .nonMatch(i -> candidate % i == 0);
}
```

### 소수로만 나누기
- 제수(divisor)를 현재 숫자 이하에서 발견한 소수로 제한할 수 있다
  - 지금까지 발견한 소수 리스트에 접근해야 한다
  - 중간 결과 리스트가 있다면 isPrime 메서드로 중간 결과 리스트를 전달하도록 다음과 같이 코드를 구현할 수 있다

```java
public static boolean isPrime(List<Integer> primes, int candidate) {
    return primes.stream().nonMatch(i -> candidate % i == 0);
}
```
- 대상 숫자의 제곱근보다 작은 소수만 사용하도록 코드를 최적화 해야 한다
  - `filter(p -> p <= candidateRoot)`를 이용해서 대상의 루트보다 작은 소수를 필터링 할 수 있다
  - 하지만 `filter`는 전체 스트림을 처리한 다음에 결과를 반환해 숫자가 클 경우 성능 문제가 발생할 수 있다
  - 대상의 제곱보다 큰 소수를 찾으면 검사를 중단함으로써 성능 문제를 없앨 수 있다
    - 정렬된 리스트와 프레디케이트를 인수로 받는 `takeWhile`을 사용할 수 있다

```java
public static boolean isPrime(List<Integer> primes, int candidate) {
    return primes.stream()
                .takeWhile(i -> i <= candidateRoot)
                .nonMatch(i -> candidate % i == 0);
}
```

### 커스텀 Collector 구현하기

**1단계: Collector 클래스 시그니처 정의**

```java
public class PrimeNumbersCollector implements Collector<Integer,
                                                         Map<Boolean, List<Integer>>,
                                                         Map<Boolean, List<Integer>>>
```

**2단계: 리듀싱 연산 구현**
```java
public Supplier<Map<Boolean, List<Integer>>> supplier() {
    return () -> new HashMap<Boolean, List<Integer>>() {{
        put(true, new ArrayList<Integer>()),
        put(false, new ArrayList<Integer>())
    }}
}

public BiConsumer<Map<Boolean, List<Integer>>, Integer> accumulator() {
    return (Map<Boolean, List<Integer>> acc, Integer candidate) -> {
        acc.get(isPrime(acc.get(true), candidate))
        .add(candidate);
    }
}
```

**3단계: 병렬 실행할 수 있는 컬렉터 만들기(가능하면)**

```java
public BinaryOperator<Map<Boolean, List<Integer>>> combiner() {
    return (Map<Boolean, List<Integer>> map1, Map<Boolean, List<Integer>> map2) -> {
        map1.get(true).addAll(map2.get(true));
        map1.get(false).addAll(map2.get(false));
        return map1;
    }
}
```
- 알고리즘 자체가 순차적이어서 컬렉터를 실제로 병렬로 사용할 수는 없다
  - `combiner`가 호출될 일이 없으므로 빈 구현을 남겨둘 수도 있다

**4단계: finisher 메서드와 컬렉터의 characteristics 메서드**

```java
public Function<Map<Boolean, List<Integer>>,
                Map<Boolean, List<Integer>>> finisher() {
    return Function.identity();                    
}
```

```java
public Function<Map<Boolean, List<Integer>>,
                Map<Boolean, List<Integer>>> characteristics() {
    return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH)));                    
}
```

