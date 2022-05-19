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
Optional<Dish> mostCalorieDish = menu.stream().collect(reducing(d1, d2 -> d1.getCalories() > d2.getCalories() ? d1 : d2));
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

