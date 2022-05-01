## 목차
- [람다 표현식](#람다-표현식)
  - [람다란 무엇인가?](#람다란-무엇인가)
  - [어디에, 어떻게 람다를 사용할까?](#어디에-어떻게-람다를-사용할까)
    - [함수형 인터페이스](#함수형-인터페이스)
    - [함수 디스크립터](#함수-디스크립터)
  - [람다 활용: 실행 어라운드 패턴](#람다-활용-실행-어라운드-패턴)
    - [1단계: 동작 파라미터화를 기억하라](#1단계-동작-파라미터화를-기억하라)
    - [2단계: 함수형 인터페이스를 이용해서 동작 전달](#2단계-함수형-인터페이스를-이용해서-동작-전달)
    - [3단계: 동작 실행](#3단계-동작-실행)
    - [4단계: 람다 전달](#4단계-람다-전달)
  - [함수형 인터페이스](#함수형-인터페이스-1)
    - [예외, 람다, 함수형 인터페이스의 관계](#예외-람다-함수형-인터페이스의-관계)
  - [형식 검사, 형식 추론, 제약](#형식-검사-형식-추론-제약)
    - [형식 검사](#형식-검사)
    - [같은 람다, 다른 함수형 인터페이스](#같은-람다-다른-함수형-인터페이스)
    - [형식 추론](#형식-추론)
    - [지역 변수 사용](#지역-변수-사용)
    - [지역 변수의 제약](#지역-변수의-제약)
  - [메서드 참조](#메서드-참조)
    - [요약](#요약)
    - [생성자 참조](#생성자-참조)
  - [람다 표현식을 조합할 수 있는 유용한 메서드](#람다-표현식을-조합할-수-있는-유용한-메서드)
    - [Comparator 조합](#comparator-조합)
    - [Predicate 조합](#predicate-조합)
    - [Function 조합](#function-조합)

# 람다 표현식
## 람다란 무엇인가?
- 람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이다
- 람다의 특징
  - 익명
    - 보통의 메서드와 달리 이름이 없으므로 익명이라 표현한다
  - 함수
    - 특정 클래스에 종속되지 않으므로 메서드가 아닌 함수이다
    - 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함한다
  - 전달
    - 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다
  - 간결성
    - 익명 클래스처럼 자질구레한 코드를 구현할 필요가 없다

```java
// 기존 코드
Comparator<Apple> byWeight = new Comparator<Apple>() {
    public int comopare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
}

// 람다를 이용한 코드
Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
- 람다 표현식은 파라미터, 화살표, 바디 세 부분으로 나뉜다

<p align=middle>
    <img width="400" alt="image" src="https://user-images.githubusercontent.com/60502370/166128611-eb840db4-efd2-479a-904e-5b6e8123d297.png">
</p>

## 어디에, 어떻게 람다를 사용할까?
- 함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다

### 함수형 인터페이스
- `Predicate<T>`가 함수형 인터페이스 중 하나이다
- 함수형 인터페이스는 오직 하나의 추상 메서드만 지정한다

```java
public interface Predicate<T> {
    boolean test(T t);
}
```
> 인터페이스는 static, default 메서드를 가질 수 있다.<br>
> 많은 다른 메서드가 있다 하더라도, 추상 메서드가 하나만 존재하면 함수형 인터페이스이다

- 람다 표현식으로 함수형 인터페이스의 추상 메서드를 구현하여 직접 전달할 수 있다
  - 즉, **전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다**
  - 기술적으로 따지면 함수형 인터페이스를 구현한 클래스의 인스턴스이다.

### 함수 디스크립터
- 람다 표현식의 시그니처를 서술하는 메서드를 함수 디스크립터라 부른다
  - `() -> void`: 파라미터 리스트가 없으며 void를 반환하는 함수
  - `(Apple, Apple) -> int`: 두 개의 Apple을 인수로 받아 int를 반환하는 함수
- `@FunctionalInterface`
  - 함수형 인터페이스임을 가리키는 어노테이션이다
  - 어노테이션을 선언했지만 실제로 함수형 인터페이스가 아닌 경우 컴파일 에러를 발생시킨다

## 람다 활용: 실행 어라운드 패턴
- 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러 싸는 형태의 코드를 **실행 어라운드 패턴**이라 한다.
- 실행 어라운드 패턴에는 try-with-resources 구문을 사용하는 것이 좋다

<p align=middle>
    <img width="400" alt="image" src="https://user-images.githubusercontent.com/60502370/166129056-a0f7c962-ad40-4fd1-8503-583c82f3b2eb.png">
</p>

### 1단계: 동작 파라미터화를 기억하라
- 현재 코드는 한 번에 한 줄만 읽을 수 있다.
- 한 번에 두 줄을 읽거나 가장 자주 사용되는 단어를 반홚하는 등 변하는 요구사항에 대응해야 할 수 있어야 한다.
- 실행, 정리 과정은 재활용하고, 다른 동작을 수행하도록 명령할 수 있다면 해결할 수 있다.
- `processFile`을 동작 파라미터화한다.

```java
// 한 줄 읽기
String result = processFile((BufferedReader br) -> br.readLine());

// 두 줄 읽기
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

### 2단계: 함수형 인터페이스를 이용해서 동작 전달
- 함수형 인터페이스 자리에 람다를 사용할 수 있다
  - `BufferedReader -> String`과 `IOException`을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어야 한다

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}

public String processFile(BufferedReaderProcessor p) throws IOException {
    ...
}
```

### 3단계: 동작 실행
```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br);
    }
}
```

### 4단계: 람다 전달
```java
// 한 줄 읽기
String result = processFile((BufferedReader br) -> br.readLine());

// 두 줄 읽기
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

## 함수형 인터페이스
<p align=middle>
    <img width="600" alt="image" src="https://user-images.githubusercontent.com/60502370/166129344-4faa9c1b-b6e2-48f4-9b1f-6b88ff1251f3.png">
</p>

### 예외, 람다, 함수형 인터페이스의 관계
- 함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다
- 예외를 던지는 람다 표현식을 만드려면 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의하거나 람다를 try/catch 블록으로 감싸야 한다
- 우리가 정의한 확인된 예외를 던지는 함수형 인터페이스는 아래와 같다  

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
```
- 하지만 `Function<T, R>` 형식의 함수형 인터페이스를 기대하는 API를 사용하면 아래와 같이 람다를 작성할 수 있다

```java
Fuction<BufferedReader, String> f = (BufferedReader b) -> {
    try {
        return b.readLine();
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```

## 형식 검사, 형식 추론, 제약
### 형식 검사
- 람다가 사용되는 컨텍스트를 이용해서 람다의 형식을 추론할 수 있다
  - 컨텍스트: 람다가 전달될 메서드 파라미터나 람다가 할당되는 변수 등
- 어떤 컨텍스트에서 기대되는 람다 표현식의 형식을 **대상 형식(target type)**이라고 부른다.

```java
List<Apple> heavierThan150g = filter(inventory, (Apple apple) -> apple.getWeight() > 150);
```

<p align=middle>
    <img width="500" alt="image" src="https://user-images.githubusercontent.com/60502370/166129821-b3cd49aa-ef49-46d4-ab1a-3231aefee818.png">
</p>

### 같은 람다, 다른 함수형 인터페이스
- **대상 형식**이라는 특징 때문에 같은 람다 표현식이더라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다
  - `Callable`과 `PribilegedAction`은 인수를 받지 않고 제네릭 형식 T를 반환하는 함수를 정의 한다

```java
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```
- 위 코드에서 첫 번째 할당문의 대상 형식은 `Callable<Integer>`고, 두 번째 할당문의 대상 형식은 `PrivilegedAction<Integer>`이다
  - 함수 디스크립터는 동일하지만 대상 형식이 다르다(?)
- 하나의 람다 표현식을 다양한 함수형 인터페이스에 사용할 수 있다

> **특별한 void 호환 규칙<br>**
> 람다의 바디에 일반 표현식이 있으면, void를 반환하는 함수 디스크립터와 호환된다. 예를 들어 List의 add 메서드는 Consumer 컨텍스트(T -> void)가 기대하는 void 대신 boolean을 반환하지만 유효한 코드이다.<br>
> Predicate<String> p = s -> list.add(s);<br>
> Consumer<String> b = s -> list.add(s);

- 할당문 컨텍스트, 메서드 호출 컨텍스트, 형변환 컨텍스트 등으로 람다 표현식의 형식을 추론할 수 있다

### 형식 추론
- 자바 컴파일러는 람다 표현식이 사용된 컨텍스트(대상 형식)를 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다
  - 대상 형식을 이용해서 함수 디스크림터를 알 수 있으므로, 컴파일러는 람다의 시그니처도 추론할 수 있다
- 컴파일러가 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다

```java
List<Apple> greenApples = filter(inventory, apple -> GREEN.equals(apple.getColor()));
```

- 상황에 따라 형식을 포함하는 것이 좋을 수도 있고, 그렇지 않을 수도 있다.
  - 가독성을 생각해서 개발자가 스스로 결정하는 것이 좋다

### 지역 변수 사용
- **람다 캡처링**
  - 람다 표현식에서는 익명 함수처럼 자유 변수(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
```

- 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 캡처할 수 있다
  - 명시적으로 final로 선언된 경우
  - 실질적으로 final로 선언된 변수와 똑같이 사용되어야 한다.

- 지역 변수는 실질적으로 final이 아니면 컴파일 에러를 뱉지만, 인스턴스 변수나 클래스 변수(static)인 경우에는 final이 아니더라도 에러를 뱉지 않는다.

**지역 변수**
<p align=middle>
    <img width="400" alt="image" src="https://user-images.githubusercontent.com/60502370/166130548-9e1c857c-ed56-4959-a7a0-46b73633b1a0.png">
<img width="592" alt="image" src="https://user-images.githubusercontent.com/60502370/166130554-47a15890-6870-424b-9b81-50797c3b4ec7.png">
</p>

- 단, 함수가 이미 실행된 후에는 변경되어도 컴파일 에러를 뱉지 않는다.

<p align=middle>
    <img width="400" alt="image" src="https://user-images.githubusercontent.com/60502370/166130579-882afd58-cf8a-40a3-b133-84d1abae7387.png">
<img width="400" alt="image" src="https://user-images.githubusercontent.com/60502370/166130588-086ad8ab-016a-4469-bf81-e1245319b46c.png">

</p>

**인스턴스 변수 / 클래스 변수**
<p align=middle>
    <img width="400" alt="image" src="https://user-images.githubusercontent.com/60502370/166130466-140d3a3d-9cd4-4c2a-a874-ff0d4f39c06b.png">
     <img width="400" alt="image" src="https://user-images.githubusercontent.com/60502370/166130485-34bfeb1c-2814-482b-af89-ce6f43f5e872.png">
</p>
   
<p align=middle>
    <img width="400" alt="image" src="https://user-images.githubusercontent.com/60502370/166130511-ea1d3d45-37c0-4ce4-9ca0-27571fbbd0c5.png"><br>
<img width="400" alt="image" src="https://user-images.githubusercontent.com/60502370/166130518-ae526331-2f98-4bbd-9131-7c497dacf083.png">
</p>

### 지역 변수의 제약
- 인스턴스 변수와 지역 변수는 태생부터 다르다
  - 인스턴스 변수는 힙에 저장된다
  - 지역 변수는 스택에 저장된다
- 자바 구현에서는 람다에 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다
  - 람다가 스레드에서 실행되고, 변수를 할당한 스레드가 사라져 변수 할당이 해제됐는데도 람다를 실행하는 스레드에서 해당 변수에 접근하려 할 수 있다
  - 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생겼다
- 인스턴스 변수나 클래스 변수 각각 힙 영역, 메소드 영역에 저장되기 때문에 여러 스레드들이 공유할 수 있다.

> **클로저(Closure)**<br>
> 클로저란 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 가리킨다. 예를 들어, 클로저를 다른 함수의 인수로 전달할 수 있다. 클로저는 클로저 외부에 정의된 변수의 값에 접근하고, 값을 바꿀 수 있다.<br>
> 자바 8의 람다와 익명 클래스는 클로저와 비슷한 동작을 수행한다. 람다와 익명 클래스 모두 메서드의 인수로 전달될 수 있으며 자신의 외부 영역의 변수에 접근할 수 있다. 다만 람다와 익명 클래스는 람다가 정의된 메서드의 지역 변수의 값은 바꿀 수 없다.

## 메서드 참조
- 메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다
- 때로는 람다 표현식보다 메서드 참조를 사용하는 것이 더 가독성이 좋고 자연스러울 수 있다

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

// java.util.Comparator.comparing
inventory.sort(comparing(Apple::getWeight));
```

### 요약
- 메서드 참조는 특정 메서드만을 호출하는 람다의 축약형이다
- 메서드 참조를 사용하면 기존 메서드 구현으로 람다 표현식을 만들 수 있다
- 메서드 명을 참조함으로써 가독성을 높일 수 있다
- 메서드 참조를 만드는 방법
  - 정적 메서드 참조
  - 다양한 형식의 인스턴스 메서드 참조
  - 기존 객체의 인스턴스 메서드 참조
    - 현존하는 외부 객체의 메서드를 호출할 때 사용한다

<p align=middle>
    <img width="500" alt="image" src="https://user-images.githubusercontent.com/60502370/166131365-7be8dbe4-bd39-430e-9c69-1e9abeaf6cdb.png">
</p>

- 컴파일러는 람다 표현식의 형식을 검사하던 방식과 비슷한 과정으로 메서드 참조가 이루어진다

### 생성자 참조
- `ClassName::new`처럼 클래스명과 new 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다

```java
// 생성자 참조
Supplier<Apple> c1 = Apple::new
Apple a1 = c1.get();

// 람다 표현식
Supplier<Apple> c1 = () -> new Apple();
Apple a1 = c1.get();
```

```java
// 생성자 참조
Function<Integer, Apple> c2 = Apple::new
Apple a2 = c2.apply(110);

// 람다 표현식
Function<Integer, Apple> c2 = (weight) -> new Apple(weight);
Apple a2 = c2.apply(110);
```

- 다음 코드는 다양한 무게를 포함하는 사과 리스트를 만든다
```java
List<Integer> weights = Arrays.asList(7, 3, 4, 10);
List<Apple> apples = map(weights, Apple::new)
public List<Apple> map(List<Integer> list, Function<Integer, Apple> f) {
    List<Apple> result = new ArrayList<>();
    for(Integer i : list) {
        result.add(f.apply(i));
    }
    return result;
}
```

- `Apple(String color, Integer weight)`처럼 두 인수를 갖는 생성자는 BiFunction 인터페이스와 같은 시그니처를 가지므로 다음과 같이 할 수 있다

```java
// 생성자 참조
BiFunction<Color, Integer, Apple> c3 = Apple::new;
Apple a3 = c3.apply(GREEN, 110);

// 람다 표현식
BiFunction<Color, Integer, Apple> c3 = (color, weight) -> new Apple(color, weight);
Apple a3 = c3.apply(GREEN, 110);
```

- 인스턴스화하지 않고도 생성자에 접근할 수 있는 기능을 다양한 상황에 응용할 수 있다
  - 예를 들어 `Map`으로 생성자와 문자열 값을 관련시킬 수 있다

```java
static Map<String, Function<Integer, Fruit>> map = new HashMap<>();
static {
    map.put("apple", Apple::new);
    map.put("orange", Orange::new);
}

public static Fruit giveMeFruit(String fruit, Integer weight) {
    return map.get(fruit.toLowerCase())
            .apply(weight);
}
```

## 람다 표현식을 조합할 수 있는 유용한 메서드
- 자바 8 API의 몇몇 함수형 인터페이스는 다양한 유틸리티 메서드를 포함한다
  - Comparator, Function, Predicate 같은 함수형 인터페이스는 람다 포현식을 조합할 수 있도록 유틸리티 메서드를 제공한다
  - 간단한 여러 개의 람다 표현식을 조합해서 복잡한 람다 표현식을 만들 수 있다는 것이다
  - 디폴트 메서드로 이러한 것들이 가능해진다

### Comparator 조합
```java
Comparator<Apple> c = Comparator.comparing(Apple::getWeight);
```

- 역정렬
  - Comparator 인터페이스에서 자체적으로 제공하는 `reversed` 메서드를 사용할 수 있다
  - `reversed`는 비교자의 순서를 뒤바꾸는 디폴트 메서드이다.

```java
inventory.sort(comparing(Apple::getWeight).reversed());
```

- Comparator 연결
  - 무게가 같은 두 사과가 존재할 때 비교 결과를 더 다듬을 수 있는 두 번째 Comparator를 만들 수 있다
  - `thenComparing`을 사용한다

```java
inventory.sort(comparing(Apple::getWeight)
        .reversed()
        .thenComparing(Apple::getCountry));
```

### Predicate 조합
- Predicate 인터페이스는 복잡한 프레디케이트를 만들 수 있도록 `negate`, `and`, `or` 세 가지 메서드를 제공한다
- `negate는` 특정 프레디케이트를 반전시킬 때 사용한다
```java
Predicate<Apple> notRedApple = redApple.negate();
```
- `and` 메서드를 이용해서 빨간색이면서 무거운 사과를 선택하도록 람다를 조합할 수 있다
```java
Predicate<Apple> redAndHeavyApple = redApple.and(apple -> apple.getWeight() > 150);
```
- `or` 메서드를 이용해서 빨간색이면서 무거운 사과 또는 그냥 녹색 사과 등 다양한 조건을 만들 수 있다
```java
Predicate<Apple> redAndHeavyApple = redApple.and(apple -> apple.getWeight() > 150)
                                            .or(apple -> GREEN.equals(apple.getColor()));
```

### Function 조합
- Function 인터페이스는 `andThen`, `compose` 두 가지 디폴트 메서드를 제공한다
- `andThen` 메서드는 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 함수를 반환한다

```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.andThen(g); // g(f(x))
int result = h.apply(1); // 4를 반환
```

- `compose` 메서드는 인수로 주어진 함수를 먼저 실행한 다음에 그 결과를 외부 함수의 인수로 제공한다

```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.compose(g); // f(g(x))
int result = h.apply(1); // 3을 반환
```
