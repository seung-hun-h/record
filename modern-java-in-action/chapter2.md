## 목차
- [동작 파라미터화 코드 전달하기](#동작-파라미터화-코드-전달하기)
  - [변화하는 요구사항에 대응하기](#변화하는-요구사항에-대응하기)
    - [1. 녹색 사과 필터링](#1-녹색-사과-필터링)
    - [2. 색을 파라미터화](#2-색을-파라미터화)
    - [3. 가능한 모든 속성으로 필터링](#3-가능한-모든-속성으로-필터링)
  - [동작 파라미터화](#동작-파라미터화)
    - [4. 추상적 조건으로 필터링](#4-추상적-조건으로-필터링)
  - [복잡한 과정 간소화](#복잡한-과정-간소화)
    - [익명 클래스](#익명-클래스)
    - [5. 익명 클래스 사용](#5-익명-클래스-사용)
    - [6. 람다 표현식 사용](#6-람다-표현식-사용)
    - [7. 리스트 형식으로 추상화](#7-리스트-형식으로-추상화)
# 동작 파라미터화 코드 전달하기
- 동작 파라미터화를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다.
  - 동작 파라미터화란 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미한다
  - 코드 블록의 실행은 나중으로 미뤄진다
- 동작 파라미터화로 수행할 수 있는 기능(?)
  - 리스트의 모든 요소에 대해서 '어떤 동작'을 수행할 수 있음
  - 리스트 관련 작업을 끝낸 다음에 '어떤 다른 동작'을 수행할 수 있음
  - 에러가 발생하면 '정해진 어떤 다른 동작'을 수행할 수 있음
- 기존 자바 API에서 동작 파라미터화를 구현하려면 쓸데 없는 코드가 늘어난다
  - 자바 8은 람다 표현식을 사용하여 코드를 간결하게 만들었다

## 변화하는 요구사항에 대응하기
### 1. 녹색 사과 필터링
```java
enum Color {
    RED, GREEN
}

public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if (GREEN.equals(apple.getColor())) {
            result.add(apple);
        }
    }
    return result;
}
```
- `inventory`에 저장된 사과 중 녹색 사과만 필터링 하는 코드
- 요구사항이 변경되어 빨간 사과만 필터링 하게 된다면, 가장 쉽게 생각할 수 있는 방법으로 메서드를 복사해서 `GREEN`을 `RED`로 바꾸는 것
- 하지만 이러한 방법은 변화하는 요구사항에 유연하게 대처할 수 없다
- **거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다** 라는 규칙은 이러한 상황에 알맞다

### 2. 색을 파라미터화
```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if (color.equals(apple.getColor())) {
            result.add(apple);
        }
    }
    return result;
}
```

- 색에 대한 변화하는 요구사항은 처리할 수 있게 되었다
- 색과 함께 무게로 사과를 필터링할 수 있도록 요구사항이 또 변경될 수 있다

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if (apple.getWeight() > weight) {
            result.add(apple);
        }
    }
    return result;
}
```
- 위 코드도 훌륭하지만, DRY(Don't repeat yourself) 원칙을 어기게 된다

### 3. 가능한 모든 속성으로 필터링
```java
public static List<Apple> filterApples(List<Apple> inventory, int weight, Color color, boolean flag) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if (flag) {
            if (color.equals(apple.getColor())) {
                result.add(apple);
            }
        } else {
            if (apple.getWeight() > weight) {
                result.add(apple);
            }
        }
        
    }
    return result;
}
```
- DRY 원칙을 위배하지 않기 위해 플래그를 사용할 수도 있다
- 하지만 플래그를 사용하는 방법은 형편없다
  - 플래그가 의미하는 바가 명확하지 않다
  - 요구사항이 변경됐을 때 유연하게 대처할 수 없다

## 동작 파라미터화
- 사과의 어떤 속성을 비교하여 불리언 값을 반환하는 방법을 사용할 수 있다

```java
public interface ApplePredicate {
    boolean test(Apple apple);
}

public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}


public class AppleGreenColortPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return GREEN.equals(apple.getColor());
    }
}
```

- 전략 패턴을 사용하여 다양한 조건으로 필터링할 수 있다
  - 전략 패턴은 전략이라 불리는 알고리즘을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법이다.
  - `ApplePredicate` 객체를 런타임에 받아와 조건을 검사한다. 이를 동작 파라미터화라 한다

### 4. 추상적 조건으로 필터링
```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if (p.test(apple)) {
            result.add(apple);
        }
    }
    return result;
}
```

- 코드/동작 전달하기
  - 코드가 유연해지고 가독성도 좋아졌다
  - 가장 중요한 부분은 `test` 메서드이다.
  - 지금은 `test`를 `ApplePredicate`로 감싸서 전달해야 한다
  - 람다를 사용하면 굳이 `ApplePredicate`로 감싸지 않고 전달할 수 있다

- 한 개의 파라미터, 다양한 동작
  - 동작 파라미터화의 강점은 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이다

![image](https://user-images.githubusercontent.com/60502370/165721870-bdeee4cd-46f7-4f04-9cb2-254582ed9682.png)

## 복잡한 과정 간소화
- `ApplePredicate`를 구현하는 여러 클래스를 정의한 다음 인스턴스화하면 번거롭고 시간이 많이 걸린다
- 자바는 클래스 선언과 인스턴스화를 동시에 수행할 수 있도록 `익명 클래스`라는 기법을 제공한다

### 익명 클래스
- 말 그대로 이름이 없는 클래스다
- 클래스 선언과 인스턴스화를 동시에 할 수 있다

### 5. 익명 클래스 사용
```java
List<Apple> redApples = filterApple(inventory, new ApplePredicate() { // 반복
    public boolean test(Apple apple) { // 반복
        return RED.equals(apple.getColor());
    } // 반복
});
```

- 익명 클래스를 사용해도 주석에 작성한 것 처럼 반복이 여전히 많다
- 많은 프로그래머들이 익명 클래스 사용에 익숙하지 않다
- 익명 클래스로 인터페이스를 구현하면 여러 클래스를 선언하는 과정을 줄일 수 있지만 여전히 장황하다
- 장황한 코드는 람다 표현식으로 해결할 수 있다

### 6. 람다 표현식 사용
```java
List<Apple> redApples = filterApple(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```
- 람다 표현식을 사용하면 코드를 간결하게 표현할 수 있다

### 7. 리스트 형식으로 추상화
```java
public interface Predicate<T> {
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for (T e : list) {
        if (p.test(e)) {
            result.add(e);
        }
    }
    return result;
}
```

- 이제 사과 뿐 아니라 바나나, 오렌지, 정수, 문자열 등의 리스트에도 메서드를 작성할 수 있다.