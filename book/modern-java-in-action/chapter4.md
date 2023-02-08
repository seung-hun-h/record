## 목차
- [스트림 소개](#스트림-소개)
  - [스트림이란 무엇인가?](#스트림이란-무엇인가)
    - [스트림을 사용하지 않는 코드](#스트림을-사용하지-않는-코드)
    - [스트림을 사용한 코드](#스트림을-사용한-코드)
    - [스트림의 소프트웨어 공학적 이득](#스트림의-소프트웨어-공학적-이득)
    - [스트림 API의 특징](#스트림-api의-특징)
  - [스트림 시작하기](#스트림-시작하기)
    - [스트림의 특징](#스트림의-특징)
  - [스트림과 컬렉션](#스트림과-컬렉션)
    - [스트림과 컬렉션의 차이](#스트림과-컬렉션의-차이)
    - [딱 한번만 탐색할 수 있다](#딱-한번만-탐색할-수-있다)
    - [외부 반복과 내부 반복](#외부-반복과-내부-반복)
  - [스트림 연산](#스트림-연산)
    - [중간 연산](#중간-연산)
    - [최종 연산](#최종-연산)

# 스트림 소개
## 스트림이란 무엇인가?
- 스트림을 이용하면 선언형으로 컬렉션 데이터를 처리할 수 있다
  - 데이터를 처리하는 임시 구현 코드 대신 질의로 표현할 수 있다
- 스트림을 이용하면 멀티스레드 코드를 구현하지 않아도 데이터를 **투명하게** 병렬로 처리할 수 있다

### 스트림을 사용하지 않는 코드
```java
List<Dish> lowCaloricDishes = new ArrayList<>();
for(Dish dish : menu) {
    if(dish.getCalories() < 400) {
        lowCaloricDishes.add(dish);
    }
}

Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
    public int compare(Dish dish1, Dish dish2) {
        return Integer.compare(dish1.getCalories(), dish2.getCalories());
    }
})

List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish dish : lowCaloricDishes) {
    lowCaloricDishesName.add(dish.getName());
}
```

### 스트림을 사용한 코드
```java
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;

List<String> lowCaloricDishesName = menu.stream()
                                        .filter(d -> d.getCalories() < 400)
                                        .sorted(comparing(Dish::getCalories))
                                        .map(Dish::getName)
                                        .collect(toList());
```

- `stream()`을 `parallelStream()`으로 바꾸면 이 코드를 멀티코어 아키텍처에서 병렬로 실행할 수 있다

```java
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;

List<String> lowCaloricDishesName = menu.parallelStream()
                                        .filter(d -> d.getCalories() < 400)
                                        .sorted(comparing(Dish::getCalories))
                                        .map(Dish::getName)
                                        .collect(toList());
```

### 스트림의 소프트웨어 공학적 이득
- 선언형으로 코드를 구현할 수 있다
  - 루프나 if문 등 제어블록을 사용하지 않고 '저칼로리의 요리만 선택하라' 같은 동작의 수행을 지정할 수 있다
- filter, sorted, map, collect 같은 여러 빌딩 블록 연산을 연결해서 복잡한 데이터 처리 파이프라인을 만들 수 있다
  - 이러한 연산들은 고수준 빌등 블록으로 이루어져있어 특정 스레딩 모델에 제한되지 않고 자유롭게 어떤 상황에서든 사용할 수 있다
  - 내부적으로 단일 스레드 모델에 사용할 수 있지만, 멀티코어 아키텍처를 최대한 투명하게 활용할 수 있게 구현되어 있다

### 스트림 API의 특징
- 선언형: 더 간결하고 가독성이 좋아진다
- 조립할 수 있음: 유연성이 좋아진다
- 병렬화: 성능이 좋아진다

## 스트림 시작하기
- 스트림은 '**데이터 처리 연산**을 지원하도록 **소스**에서 추출된 **연속된 요소**'로 정의 할 수 있다
- 연속된 요소
  - 컬렉션과 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공한다
  - 컬렉션은 자료구조이므로 시간과 공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룬다
  - 스트림은 filter, sorted, map처럼 계산식이 주를 이룬다
- 소스
  - 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다
  - 리스트로 스트림을 만들면 스트림의 요소는 리스트 요소와 같은 순서를 유지한다
- 데이터 처리 연산
  - 스트림은 함수형 프로개르밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산을 지원한다
  - 스트림 연산은 순차적으로 또는 병렬로 실행할 수 있다

### 스트림의 특징
- 파이프라이닝
  - 대부분의 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다
  - 게으름과 쇼트서킷 같은 최적화도 얻을 수 있다
- 내부 반복
  - 스트림은 컬렉션과 달리 내부 반복을 지원한다

## 스트림과 컬렉션
- 자바의 기존 컬렉션과 스트림은 모두 **연속된** 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다
  - **연속된**이란 표현은 순서와 상관없이 아무 값에나 접속하는 것이 아니라 순차적으로 접근한다는 것을 의미한다

### 스트림과 컬렉션의 차이
- 데이터 계산 시점
  - 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 한다
    - 컬렉션은 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료구조다
  - 스트림은 이론적으로 요청할 때만 요소를 계산하는 고정된 자료구조다
    - 사용자가 요청하는 값만 스트림에서 추출하는 것이 핵심이다
    - 사용자 입장에서 변화를 알 수 없다. 
    - 결과적으로 스트림은 생산자와 소비자 관계를 형성한다
  - 컬렉션은 적극적으로 생성된다
    - 요소가 모두 계산된 후 저장된다
  - 스트림은 게으르게 만들어지는 컬렉션과 같다
    - 사용자가 요청할 때만 값을 계산한다

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/167357873-380e507f-f633-42c6-bd8a-e083e4c1b099.png">

### 딱 한번만 탐색할 수 있다
- 반복자(Iterator)와 마찬가지로 스트림도 한 번만 탐색할 수 있다
  - 탐색된 스트림은 소비된다
- 한 번 탐색한 요소를 다시 탐색하기 위해서 새로운 스트림을 만들어야 한다
> **스트림과 컬렉션의 철학적 접근**<br>
> 스트림은 시간적으로 흩어진 값의 집합으로 간주할 수 있다<br>
> 반면 컬렉션은 특정 시간에 모든 것이 존재하는 공간에 흩어진 값으로 비유할 수 있다

### 외부 반복과 내부 반복
- 컬렉션 인터페이스를 사용하기 위해서는 외부 반복을 사용한다
  - 외부 반복은 사용자가 직접 요소를 반복하는 것(for-each)을 의미한다

```java
List<String> names = new ArrayList<>();
for(Dish dish : menu) {
  names.add(dish.getName());
}

// Iterator 사용
List<String> names = new ArrayList<>();
Iterator<String> iterator = menu.iterator();
while(iterator.hasNext()) {
  Dish dish = interator.next();
  names.add(dish.getName());
}
```

- 스트림 라이브러리는 내부 반복을 사용한다.
  - 함수에 어떤 작업을 수행할지만 지정하면 모든 것이 알아서 처리된다.

```java
List<String> names = menu.stream()
                        .map(Dish::getName)
                        .collect(toList());
```
- 내부 반복의 장점
  - 작업을 투명하게 병렬로 처리할 수 있다
  - 작업을 더 최적화된 다양한 순서로 처리할 수 있다
  - 스트림 라이브러리는 데이터 표현과 하드웨어를 활용한 병렬성 구현을 자동으로 선택한다

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/167359383-ffbb63c9-674c-4223-a4e9-cd680d0e381b.png">

## 스트림 연산
<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/167359591-b9b6206e-c82c-4ede-b0e2-fe5d01071c59.png">

- 중간 연산: 연결할 수 있는 스트림 연산
- 최종 연산: 스트림을 닫는 연산

### 중간 연산
- `filter`, `sorted` 같은 중간 연산은 다른 스트림을 반환하여 여러 중간 연산을 연결할 수 있다
- 중간 연산의 가장 중요한 특징은 **게으르다(Lazy)**는 것이다
  - 중간 연산을 합친 다음에 중간 연산을 최종 연산으로 한 번에 처리한다(?)
- 모든 스트림 요소를 처리하지 않고도 결과를 반환할 수 있다(쇼트 서킷)
  - `limit`은 스트림을 반환하므로 중간연산
  - `allMatch`, `findAny`은 스트림을 반환하지 않으므로 최종연산
  - 둘 다 쇼트 서킷 기법을 적용한다
- 서로 다른 연산들이 한 과정으로 병합된다(루프 퓨전)
  - `filter`, `map` 같은 서로 다른 연산이 병합된다
  - 스트림을 사용하지 않으면 filter와 map 연산이 서로 다른 루프 안에서 이루어져야할 수 있지만, 한 번의 루프에서 해결이 가능하다(?)

### 최종 연산
- 최종 연산은 스트림 파이프라인에서 결과를 도출한다
- 최종 연산에 의해 List, Integer, void 등 스트림 이외의 결과가 반환된다.