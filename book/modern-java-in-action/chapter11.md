# null 대신 Optional 클래스

## 값이 없는 상황을 어떻게 처리할까?
- 예제 코드
```java
@Getter
public class Person {
    private Car car;
}

@Getter
public class Car {
    private Insurance insurance;
}

@Getter
public class Insurance {
    private String name;
}
```

```java
public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
}
```

- 위 코드는 `NullPointerException`이 발생할 수 있다
  - 어떤 사람은 자동차를 가지지 않을 수도 있다
  - 어떤 사람은 자동차의 보험을 가입하지 않을 수도 있다

### 보수적인 자세로 NullPointerException 줄이기
- 필요한 곳에 null 검사를 할 수 있다
  - 더 보수적인 개발자라면 사용하지 않는 곳에서도 null 검사를 할 것이다

```java
public String getCarInsuranceName(Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if(insurance != null) {
                return insurance.getName();
            }
        }
    }

    return "Unknown";
}
```

- 위 코드처럼 모든 변수가 null인지 의심 하므로 if 구문이 늘어나며 복잡도가 증가한다
- 이와 같은 반복 패턴(Recurring Pattern) 코드를 깊은 의심(Deep Doubt)라고 부른다

```java
public String getCarInsuranceName(Person person) {
    if (person == null) 
        return "Unknown";

    Car car = person.getCar();
    if (car == null) 
        return "Unknown";

    Insurance insurance = car.getInsurance();
    if (insurance == null) 
        return "Unknown";

    return insurance.getName();
}
```

- null인 경우 즉시 반환 하도록 코드를 변경했다
  - 출구(반환)이 네 가지가 되므로 그리 좋은 코드는 아니다

### null 때문에 발생하는 문제
- 에러의 근원이다
  - `NullPointerException`은 자바에서 가장 흔히 발생하는 에러이다
- 코드를 어지럽힌다
  - 중첩된 null 확인 코드를 추가해야 하므로 가독성이 떨어질 수 있다
- 아무 의미가 없다
  - null은 아무 의미가 없다
  - 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다
- 자바 철학에 위배된다
  - 자바는 개발자에게 모든 포인터를 숨겼지만, null은 그러하지 못했다
- 형식 시스템에 구멍을 만든다
  - null은 무형식이다
  - 모든 참조 형식에 null을 할당할 수 있다
  - null이 시스템의 다른 부분에 퍼졌을 때 null이 어떤 의미로 사용되었는지 확인할 수 없다

### 다른 언어는 null 대신 무엇을 사용하나?
- 최근 그루비 같은 언어에서는 안전 내비게이션 연산자(Safe Navigation Operator)인 `?.`을 도입해서 null 문제를 해결했다
  - 그루비 안전 내비게이션 연산자를 사용하면 null 참조 걱정 없이 객체에 접근할 수 있다

```groovy
def carInsuranceName = person?.car?.insurance?.name
```

- 하스켈이나 스칼라 등의 함수형 언어는 아예 다른 관점에서 null 문제를 접근한다
  - 하스켈은 선택형 값(Optional Value)를 저장할 수 있는 `Maybe`라는 형식을 제공한다
    - `Maybe`는 주어진 형식의 값을 갖거나 아니면 아무 값도 갖지 않을 수 있다
  - 스칼라도 T 형식의 값을 갖거나 아무것도 가지지 않을 수 있는 `Option[T]`라는 구조를 제공한다
    - `Option` 형식은 값이 있는지 여부를 명시적으로 확인해야하므로, null과 관련한 문제가 발생할 가능성이 적다

> 그렇다면 메서드를 작성할 때 null을 반환해서는 안되는가?

## Optional 클래스 소개
- 자바 8은 하스켈과 스칼라의 영향을 받아서 `java.util.Optional<T>`라는 새로운 클래스를 제공한다

```java
@Getter
public class Person {
    private Optional<Car> car;
}

@Getter
public class Car {
    private Optional<Insurance> insurance;
}

@Getter
public class Insurance {
    private String name; // 보험회사는 반드시 이름을 가진다
}
```

> 우리가 흔히 작성하는 Model 클래스에 Optional을 사용하는 것은 어떤가?
> 차라리 final을 사용하면 어떤가? 기본 생성자를 사용핳지 못하는 문제점은 있곘다

- `Optional`을 사용하면 의미가 더 명확해질 수 있다
- 보험사 이름은 `Optional`을 사용하지 않으므로 반드시 가져야할 속성임을 알 수 있다
- 보험사의 이름이 존재하지 않는다면 예외처리 코드를 작성하는 것이 아니라, 문제의 원인을 찾아 문제를 해결해야한다
- `Optional`을 이용하면 값이 없는 상황이 우리 데이터에 문제가 있는 것인지, 아니면 알고리즘의 버그인지 명확하게 구분할 수 있다
- 모든 null 참조를 `Optional`로 대치하는 것은 바람직하지 않다
  - `Optional`의 역할은 더 이해하기 쉬운 API를 설계하도록 돕는 것이다.
  - `Optional`이 등장하면 값이 없을 수 있는 상황에 적절하게 대응하도록 강제하는 효과가 있다

## Optional 적용 패턴
### Optional 객체 만들기

**빈 Optional**
```java
Optional<Car> optCar = Optional.empty();
```

**null이 아닌 값으로 Optional 만들기**
```java
Optional<Car> optCar = Optional.of(car);
```

- car가 null이라면 즉시 `NullPointerException`이 발생한다

**null 값으로 Optional 만들기**
```java
Optional<Car> optCar = Optional.ofNullable(car);
```
- car가 null이면 빈 Optional 객체가 반환된다

### 맵으로 Optional의 값을 추출하고 변환하기
- 보동 객체의 정보를 추출하는 경우가 많은데, `Optional`을 사용하지 않으면 if 구문을 사용해서 null 검사를 해야한다.
- `Optional`는 이를 위해서 `map`을 제공한다

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

- 값이 있으면 변환해주고, 비어있으면 아무일도 일어나지 않는다

### flatMap으로 Optional 객체 연결

```java
public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
}
```

- 위처럼 여러 메서드를 호출해야 하는 경우에 `map`을 사용하면 아래와 같다

```java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name =
    optPerson.map(Person::getCar)
            .map(Car::getInsurance)
            .map(Insurance::getName);
```

- 하지만 위 코드는 컴파일되지 않는다
  - `Optional<Person>`은 map을 호출할 수 있지만, `getCar`는 `Optional<Car>`을 반환하므로 최종적으로 `Optional<Optional<Car>>`와 같은 형식이된다
- `flatMap`을 사용하면 위 문제를 해결할 수 있다
  - `flatMap`을 사용하면 이차원 `Optional`을 일차원 `Optional`로 평준화할 수 있다

```java
public String getCarInsuranceName(Optional<Person> person) {
    return person.flatMap(Person::getCar)
                .flatMap(Car::getInsurance)
                .map(Insurance::getName)
                .orElse("Unknown");
}
```

- `Optional`을 사용하면 null 검사를 위한 분기가 발생하지 않는다
- 그리고 도메인 모델과 관련한 암묵적인 지식에 의존하지 않고 명시적으로 형식 시스템을 정의할 수 있다
- `Optional`을 반환하거나 `Optional`을 반환하는 메서드를 정의한다면 결과적으로 이 메서드를 사용하는 모든 사람에게 이 메서드가 빈 값을 받거나 빈 값을 반환할 수 있음을 잘 문서화하는 것과 같다

**도메인 모델에 Optional을 사용했을 때 데이터를 직렬화 할 수 없는 이유**
- 자바 언어 아키텍트인 브라이언 고츠는 `Optional`의 용도가 선택형 반환값을 지원하는 것이라고 명확하게 못박았다
  - 따라서 직렬화를 위한 인터페이스인 `Serializable` 인터페이스를 구현하지 않았다
- 모델에 `Optional` 클래스를 필드 형식으로 사용하면 프레임워크나 라이브러리를 사용할 때 문제가 발생할 수 있다
  - 그럼에도 선택형 값에는 Optional을 사용하는 것이 좋다고 생각한다(저자가)
  - 필드에 직접 사용하지 않고 반환 값에 사용할 수 있겠다

```java
public class Person {
    private Car car;
    public Optional<Car> getCarAsOptional() {
        return Optional.ofNullable(car);
    }
}
```

### Optional 스트림 조작
- `Optional`은 `stream()` 메서드를 제공한다
  - 값이 존재하면 `Stream.of(value)`
  - 값이 존재하지 않으면 `Stream.empty()`

```java
people.stream()
        .map(Person::getCar)
        .map(optCar -> optCar.flatMap(Car::getInsurance))
        .map(optInsurance -> optInsurance.map(Insurance::getName))
        .flatMap(Optional::stream)
        .collect(Collectors.toSet());
```

- 위 처럼 `Optional`의 값이 존재하는지 여부와는 상관없이 코드를 작성할 수 있다

### 디폴트 액션과 Optional 언랩
- `get()`
  - 값을 읽는 가장 간단한 메서드면서 동시에 가장 안전하지 않는 메서드이다
  - 값이 없는 경우 `NoSuchElementException`을 발생시킨다
  - 값이 존재하는 것이 확실하지 않는 상황에서는 사용하지 않는 것이 좋다(거의 사용하지 않는 듯)

- `orElse(T other)`
  - 값이 존재하지 않을 때 기본값을 제공할 수 있다
```java
public T orElse(T other) {
    return value != null ? value : other;
}
```

- `orElseGet(Supplier<? extends T> other)`
  - `orElse`에 대응하는 게으른 버전의 메서드이다
  - Optional에 값이 없을때만 Supplier가 동작한다
  - 기본값이 반드시 필요한 경우에 사용한다

- `orElseThrow(Supplier<? extends X> exceptionSupplier)`
  - Optional이 비어있을때 예외를 발생시킨다

- `ifPresent(Consumer<? super T> consumer)`
  - 값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다
  - 값이 없으면 아무일도 일어나지 않는다

- `ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`
  - Optional이 비어있을 때 실행할 수 있는 Runnable을 인수로 받는다는 점만 `ifPresent`와 다르다

### 두 Optional 합치기
- Person과 Car 정보를 이용해서 가장 저렴한 보험료를 제공하는 보험회사를 찾는 로직

```java
public Insurance findCheapestInsurance(Person person, Car car) {
  // 다양한 보험회사가 제공하는 서비스 조회
  // 모든 결과 데이터 비교
  return cheapestCompany;
}
```

- null safe 확인 코드

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
  if (person.isPresent() && car.isPresent()) {
    return Optional.of(findCheaptestInsurance(person.get(), car.get()));
  }
  return Optional.empty();
}
```
- null을 직접 확인하는 것과 다른점이 없다
- `flatMap()`을 사용하면 한 줄의 코드로 리팩토링할 수 있다

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
  return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```

- `person.flatMap`은 값이 비어있다면 아무런 값도 전달하지 않고, 값이 존재하면 인자로 전달한 람다를 실행한다
- `car.map`은 값이 비어있다면 빈 Optional을 반환하고, 값이 존재한다면 인자로 전달한 람다를 실행한다.

### 필터로 특정값 거르기
- `Optional`은 `filter`를 제공하여 객체의 특정 프로퍼티를 검사할 수 있다

```java
optInsurance.filter(insurance -> 
                      "CambridgeInsurance".equals(insurance.getName()))
            .ifPresent(x -> System.out.println(x));
```

- Optional 값이 비어있다면 `filter` 연산은 아무 동작 하지 않는다.
- `filter`의 결과가 false인 경우에는 값은 사라지고 Optional은 빈 상태가 된다

## Optional을 사용한 실용 예제
### 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기
- 반환값이 잠재적으로 null이 될 수 있다면 Optional로 감싸서 반환하는 것이 바람직하다

> Optional이 아닌 반환값은 반드시 null이 아니거나 아니어야한다고 서로간의 약속이 되어 있으면 좋겠다
> 설령 null이더라도 null이 반환된 것이 비정상적인 로직의 결과라는 것이 드러나지 않을까

```java
Optional<Object> value = Optional.ofNullabe(map.get("key"));
```

### 예외와 Optional 클래스
- 예외 상황에서 빈 Optional을 반환할 수도 있다
- `Integer.parseInt`는 인자를 변환할 수 없는 경우 `NumberFormatException`을 발생시킨다
  - 이때 try/catch 문을 사용해서 빈 Optional을 반환할 수 있다
  - Optional 유틸 클래스를 작성하면 매번 try/catch 문을 작성하지 않아도 된다

```java
public static Optional<Integer> stringToInt(String s) {
  try {
    return Optional.of(Integer.parseInt(s));
  } catch (NumberForfatException e) {
    return Optional.empty();
  }
}
```

### 기본형 Optional을 사용하지 말아야 하는 이유
- OptionalInt, OptionalLong, OptionalDouble 등 기본형 특화 Optional 클래스가 존재한다
- Stream에서는 요소의 수가 많을 수 있기 때문에 기본형 특화 클래스를 사용하여 성능을 개선할 수 있다
- 하지만 Optional은 요소 수가 하나로 정해져있기 떄문에 기본형 특화 클래스로 성능을 개선할 수 없다
- 기본형 특화 Optional을 사용하는 것은 권장하지 않는다