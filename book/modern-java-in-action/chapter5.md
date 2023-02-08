## 목차
- [스트림 활용](#스트림-활용)
  - [필터링](#필터링)
    - [프레디케이트로 필터링](#프레디케이트로-필터링)
    - [고유 요소 필터링](#고유-요소-필터링)
  - [스트림 슬라이싱](#스트림-슬라이싱)
    - [프레디케이트를 이용한 슬라이싱](#프레디케이트를-이용한-슬라이싱)
    - [요소 건너뛰기](#요소-건너뛰기)
  - [매핑](#매핑)
    - [스트림의 각 요소에 함수 적용하기](#스트림의-각-요소에-함수-적용하기)
    - [스트림 평면화](#스트림-평면화)
  - [검색과 매칭](#검색과-매칭)
    - [프레디케이트가 적어도 한 요소와 일치하는지 확인](#프레디케이트가-적어도-한-요소와-일치하는지-확인)
    - [프레디케이트가 모든 요소와 일치하는지 검사](#프레디케이트가-모든-요소와-일치하는지-검사)
    - [요소 검색](#요소-검색)
    - [첫 번째 요소 찾기](#첫-번째-요소-찾기)
  - [리듀싱](#리듀싱)
    - [요소의 합](#요소의-합)
    - [최대값과 최소값](#최대값과-최소값)
    - [reduce 메서드의 장점과 병렬화](#reduce-메서드의-장점과-병렬화)
  - [스트림 연산: 상태 없음과 상태 있음](#스트림-연산-상태-없음과-상태-있음)
    - [stateless operation](#stateless-operation)
    - [stateful operation](#stateful-operation)
  - [숫자형 스트림](#숫자형-스트림)
    - [기본형 특화 스트림](#기본형-특화-스트림)
    - [숫자 범위](#숫자-범위)
  - [스트림 만들기](#스트림-만들기)
    - [값으로 스트림 만들기](#값으로-스트림-만들기)
    - [null 이 될 수 있는 객체로 스트림 만들기](#null-이-될-수-있는-객체로-스트림-만들기)
    - [배열로 스트림 만들기](#배열로-스트림-만들기)
    - [파일로 스트림 만들기](#파일로-스트림-만들기)
    - [함수로 무한 스트림 만들기](#함수로-무한-스트림-만들기)

# 스트림 활용
## 필터링
### 프레디케이트로 필터링
- `filter`는 **프레디케이트**를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 **스트림**을 반환한다
  - 프레디케이트는 불리언을 반환하는 함수이다
- 모든 채식 요리를 필터링하는 코드
```java
List<Dish> vegetarianMenu = menu.stream()
                                .filter(Dish::isVegetarian)
                                .collect(toList());
```

### 고유 요소 필터링
- `distinct`는 고유 요소로 이루어진 스트림을 반환한다
  - 고유 여부는 스트림에서 만든 객체의 `hashCode`, `equals`로 결정된다
  - 고유 요소란 유일한 요소를 의미한다
- 리스트의 모든 짝수를 선택하고 중복을 필터링하는 코드
```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
        .filter(i -> i % 2 == 0)
        .distinct()
        .forEach(System.out::println);
```

## 스트림 슬라이싱
### 프레디케이트를 이용한 슬라이싱
- **자바 9**는 스트림의 요소를 효과적으로 선택할 수 있도록 `takeWhile`, `dropWhile` 두 가지 새로운 메서드를 제공한다

**takeWhile**

- 무한 스트림을 포함한 모든 스트림에 프레디케이트를 적용해 스트림을 슬라이스 할 수 있다
- 스트림의 요소들 중 프레디케이트를 만족하지 않는 최초 지점의 이전 스트림을 반환한다
  - 정렬된 상태의 리스트에서 사용할 수 있다
  - 정렬되지 않은 상태에서 동작은 하지만 원하는 결과를 얻을 수 없다
- [참고](https://www.geeksforgeeks.org/stream-takewhile-method-in-java-with-examples/)

> **parallel 상태에서도 takeWhile은 정상동작할까?**<br>
>  정상 동작한다!<br>

```java
//given
List<Integer> numbers = new ArrayList<>();
for (int i=0;i<50;i++) {
    numbers.add(i);
}

//when
List<Integer> collect = numbers.stream()
    .parallel()
    .filter(number -> number % 2 == 0)
    .takeWhile(number -> number < 21)
    .toList();

//then
System.out.println("collect = " + collect);

//collect = [0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

**dropWhile**

- `takeWhile`과 반대로 프레디케이트가 처음으로 거짓이되는 지점까지 요소를 버린다

### 요소 건너뛰기
- 스트림은 처음 n개 요소를 제외한 스트림을 반환하는 `skip(n)` 메서드를 지원한다
  - n개 이하의 요소를 포함하는 스트림에 `skip(n)`을 호출하면 빈 스트림이 반환된다
- `parallelStream()`에서도 잘 동작한다

## 매핑
### 스트림의 각 요소에 함수 적용하기
- `map`은 함수를 인수로 받는다
- 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑된다

### 스트림 평면화
- `map`을 사용해서 리스트에서 고유 문자로 이루어진 리스트를 반환하는 코드
  - ["Hello", "World"]를 ["H", "e", "l", "o", "W", "r", "d"]로 변환

```java
words.stream()
    .map(word -> word.split(""))
    .distinct()
    .collect(toList());

// [H, e, l, l, o]
// [W, o, r, l, d]
```

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/167730758-13ec4a72-1b44-46d1-b0c5-5956d8ee67f0.png">

- 위 코드에서 `map`으로 전달한 람다는 각 단어의 String[]을 반환한다는 점이 문제이다
- `map`과 `Array.stream`을 사용하여 해결을 시도해볼 수 있다

```java
words.stream()
    .map(word -> word.split(""))
    .map(Arrays::stream)
    .distinct()
    .collect(toList());
```
- 하지만 위 코드는 스트림 리스트(`List<Stream<String>>`)이 반환되며 문제가 해결되지 않는다
- `flatMap`을 사용하면 문제를 해결할 수 있다
  - `flatMap`은 각 배열을 스트림이 아니라 스트림의 컨텐츠로 매핑한다
  - `map(Arrays::stream)`과 달리 하나의 평면화된 스트림을 반환한다
```java
words.stream()
    .map(word -> word.split(""))
    .flatMap(Arrays::stream)
    .distinct()
    .collect(toList());
```

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/167732213-2676f0ad-6225-45ff-957f-f27df9b57982.png">

## 검색과 매칭
### 프레디케이트가 적어도 한 요소와 일치하는지 확인
- 프레디케이트가 주어진 스트림에서 적어도 한 요소와 일치하는지 확인할 때 `anyMatch`를 사용한다
- `anyMatch`는 불리언을 반환하므로 최종 연산이다.

### 프레디케이트가 모든 요소와 일치하는지 검사
- `allMatch`는 스트림의 모든 요소가 주어진 프레디케이트와 일치하는지 검사한다
- `allMatch`의 반대 연산으로는 `noneMatch`가 있다
- `anyMatch`, `allMatch`, `allMatch` 세 가지 메서드는 쇼트 서킷 기법을 사용한다
  - 모든 스트림의 연산을 처리하지 않고도 반환할 수 있다

### 요소 검색
- `findAny` 메서드는 현재 스트림에서 임의의 요소를 반환한다
  - `Optional`을 반환하므로 다른 스트림연산과 연결해서 사용할 수 없다
  - `Optional`은 값이 존재하는지 확인하고 값이 없을 때 어떻게 처리할지 강제하는 기능을 제공한다

### 첫 번째 요소 찾기
- 논리적인 아이템 순서가 정해져있는 경우, 첫 번째 요소를 찾기 위해 `findFirst`를 사용한다

> **findFirst, findAny 언제 사용하나?**<br>
> 병럴성 때문에 두 메서드가 존재한다<br>
> 병럴 실행에서는 첫 번째 요소를 찾기 어렵다. 따라서 반환 순서가 상관 없다면 병렬 스트림에서는 `findAny`를 주로 사용한다.

## 리듀싱
- 리듀싱 연산은 모든 스트림 요소를 처리해서 값으로 도출하는 연산이다
- 함수형 프로그래밍 언어 용어로는 이 과정이 마치 종이를 작은 조각이 될 때까지 반복해서 접는 것과 비슷하다는 의미로 폴드라고 부른다

### 요소의 합
```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
```

- 초기값 0
- 두 요소를 조합해서 새로운 값을 만드는 `BinaryOperator<T>`

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/167737551-b58f5094-5f9e-4b33-9464-bbf5f2ab55c4.png">

- a는 누적 값, b는 새로운 요소 값이다
- 초기값이 없는 경우에는 `Optional`을 반환한다.
  - 스트림에 아무요소가 없는 경우 초기값이 없으므로 reduce의 합계를 반환할 수 없기 때문이다

### 최대값과 최소값
```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> max = numbers.stream().reduce(Integer::min);
```

### reduce 메서드의 장점과 병렬화
- `reduce`를 이용하면 내부 반복이 추상화되면서 내부 구현에서 병렬로 `reduce`를 실행할 수 있다
- 외부 반복을 사용하면 공유 변수인 `sum`을 이용해야 하므로 병렬 실행이 쉽지않다

## 스트림 연산: 상태 없음과 상태 있음

### stateless operation
- `map`, `filter` 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보낸다
  - 사용자가 제공한 람다나 메서드 참조가 내부적인 가변 상태를 갖지 않는다는 가정하에
  - 이들은 보통 상태가 없는 즉, 내부 상태를 갖지 않는(stateless operation) 연산이다.

### stateful operation
- `reduce`, `sum`, `max` 같은 연산은 결과를 누적할 내부 상태가 필요하다
  - 스트림에서 처리하는 요소 수와 관계 없이 내부 상태의 크기는 한정(Bounded)되어 있다.(?)
    - `int`, `double`을 사용하므로 그 크기가 한정되어 있다는 말인 것 같다

- `sorted`, `distinct` 같은 연산은 filter나 map과 다르게 과거 이력을 알고 있어야 한다
  - 어떤 요소를 출력 스트림으로 추가하려면 모든 요소가 버퍼에 추가되어 있어야 한다(?)
  - 연산에 필요한 저장소 크기가 정해져있지 않아(Unbounded) 데이터 크기가 크거나 스트림이 무한이면 문제가 발생할 수 있다
  - 따라서 내부 상태를 가진다

## 숫자형 스트림
```java
int calories = menu.stream()
                    .map(Dish::getCalories)
                    .reduce(0, Integer::sum);
```
- 위 코드는 박싱 비용이 숨겨져있다.
- 스트림 API는 숫자 스트림을 효율적으로 처리할 수 있도록 **기본형 특화 스트림**을 제공한다

### 기본형 특화 스트림
- IntStream
  - `mapToInt()`
  - IntStream의 map 연산은 `IntUnaryOperator`를 인수로 받는다
  - `IntUnaryOperator`는 int를 인수로 받아서 int를 반환하는 람다이다
  - IntStream에서 정수가 아닌 다른 값으로 변환하고 싶을 때는 `boxed()`를 사용한다
  - `boxed()`는 특화 스트림을 일반 스트림으로 변환한다
```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
Stream<Integer> stream = intStream.boxed();
```
  - `OptionalInt`
    - 스트림에 요소가 없는 상황과 실제 최대값이 0인 상황을 구분하기 위해 존재
    - `Optional`을 기본형 특화한 것이다
- DoubleStream
- LongStream

### 숫자 범위
- IntStream, LongStream에서 `range`, `rangeClosed` 두 가지 정적 메서드를 제공한다
- `range` 메서드는 시작값과 종료값이 결과에 포함되지 않는다
- `rangeClosed` 메서드는 시작값과 종료값이 결과에 포함된다

```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100) // [1, 100]
                                .filter(n -> n % 2 == 0);
```

## 스트림 만들기
### 값으로 스트림 만들기
- `Stream.of()`

```java
Stream<String> stream = Stream.of("Modern", "Java", "In", "Action");
```
- `Stream.empty()`

```java
Stream<String> stream = Stream.empty();
```

### null 이 될 수 있는 객체로 스트림 만들기
- java 9부터 null이 될 수 있는 개체로 스트림을 만들 수 있는 새로운 메서드가 추가되었다

```java
// java 8 코드
String homeValue = System.getProperty("home");
Stream<String> homeValueStream = homeValue == null ? Stream.empty() : Stream.of(value);

// java 9 코드
Stream<String> homeValue = Stream.ofNullable(System.getProperty("home"));
```

- `flatMap()`을 활용한 패턴

```java
Stream<String> values = Stream.of("config", "home", "user")
                            .flatMap(key -> Stream.ofNullable(System.getProperty(key)));
```

### 배열로 스트림 만들기
```java
int[] numbers = {2, 3, 4, 5, 6, 7, 8, 9};
int sum = Arrays.stream(numbers).sum();
```

### 파일로 스트림 만들기

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/168460606-1d33bdcf-63ed-4459-a787-906963deb2a0.png">
- Stream은 AutoCloseable 이므로 try-with-resource 사용가능

### 함수로 무한 스트림 만들기
- `Stream.iterate()`, `Stream.generate()`
- 두 연산을 이용해서 무한 스트림을 만들 수 있다
- 일반적으로 `limit(n)`을 함께 사용한다

**iterate 메서드**

```java
Stream.iterate(0, n -> n + 2)
      .limit(10)
      .forEach(System.out::println);
```
- `iterate()`는 초기값과 람다를 인수로 받아서 새로운 값을 끊임없이 생산할 수 있다
- 요청할 때마다 값을 생산할 수 있고, 끝이 없으므로 `언바운드 스트림`이다.
- java 9에서 `iterate()`는 프레디케이트를 지원한다

```java
IntStream.iterate(0, n -> n < 100, n -> n + 4)
        .forEach(System.out::println);
```
- 두 번째 인수로 프레디케이트를 받아 언제까지 작업을 수행할 것인지 기준으로 사용한다

**generate 메서드**
- `iterate()`와 달리 생산된 값을 연속적으로 계산하지 않는다
- `Supplier<T>`를 인수로 받아서 새로운 값을 생산한다

```java
Stream.generate(Math::random)
      .limit(5)
      .forEach(System.out::println);
```

- 위 코드에서는 Supplier가 상태를 저장하지 않는다
- 하지만 구현에 따라서 상태를 가질 수도 있다
- Supplier가 상태를 가질 경우 병렬 코드에서는 안전하지 않다는 것을 명심해야 한다
- 상태를 가진 Supplier는 단지 설명에 필요한 예제일 뿐 실제로는 피해야 한다

```java
// stateless code
IntStream ones = IntStream.generate(() -> 1);
IntStream twos = IntStream.generate(() -> 2);
IntStream twos = IntStream.generate(new IntSupplier() {
  public int getAsInt() {
    return 1;
  }
});


// stateful code
IntSupplier fib = new IntSupplier() {
  private int previous = 0;
  private int current = 1;
  public int getAsInt() {
    int oldPrevious = this.previous;
    int nextValue = this.previous + this.current;
    this.previous = this.current;
    this.current = nextValue;
    return oldPrevious;
  }
};

IntStream.generate(fib).limit(10).forEach(System.out::println);
```
- `fib`는 가변 상태 객체이다
- 스트림을 병렬로 처리하면서 올바른 결과를 얻으려면 불변 상태 기법을 고수해야 한다.