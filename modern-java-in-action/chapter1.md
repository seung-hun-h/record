## 목차
- [CHAPTER1 자바 8, 9, 10, 11: 무슨일이 일어나고 있는가?](#chapter1-자바-8-9-10-11-무슨일이-일어나고-있는가)
  - [역사의 흐름은 무엇인가?](#역사의-흐름은-무엇인가)
    - [자바 8에서 제공하는 새로운 기술](#자바-8에서-제공하는-새로운-기술)
  - [왜 아직도 자바는 변화하는가?](#왜-아직도-자바는-변화하는가)
    - [프로그래밍 언어 생태계에서 자바의 위치](#프로그래밍-언어-생태계에서-자바의-위치)
    - [스트림 처리](#스트림-처리)
    - [동작 파라미터화로 메서드에 코드 전달하기](#동작-파라미터화로-메서드에-코드-전달하기)
    - [병렬성과 공유 가변 데이터](#병렬성과-공유-가변-데이터)
    - [자바가 진화해야 하는 이유](#자바가-진화해야-하는-이유)
  - [자바 함수](#자바-함수)
    - [메서드와 람다를 일급 시민으로](#메서드와-람다를-일급-시민으로)
  - [스트림](#스트림)
    - [멀티스레딩은 어렵다](#멀티스레딩은-어렵다)
  - [디폴트 메서드와 자바 모듈](#디폴트-메서드와-자바-모듈)
  - [함수형 프로그래밍에서 가져온 다른 유용한 아이디어](#함수형-프로그래밍에서-가져온-다른-유용한-아이디어)

# CHAPTER1 자바 8, 9, 10, 11: 무슨일이 일어나고 있는가?
## 역사의 흐름은 무엇인가?
자바 역사를 통틀아 가장 큰 변화가 자바 8에서 일어 났다.
- 코드의 변화
  - 람다 표현식, 메서드 참조
- 멀티 코어 CPU 대중화 같은 하드웨어적인 변화
  - 자바 8이전까지 나머지 코어(멀티 코어 CPU에서)를 사용하기 위해 스레드를 사용하기를 권장했으나, 전문가를 제외하고는 사용하기 어려웠다
  - 자바 1.0에서 스레드와 락, 메모리 모델까지 지원했지만, 팀원 전체가 저수준 기능을 모두 활용하기는 어려웠다
  - 자바 5에서 스레드 풀, 병렬 실행 커넥션 등 기능 제공, 자바 7에서 포크/조인 프레임워크 제공 했지만 여전히 개발자가 사용하기 어려웠다
  - 자바 8에서 새로운 기법을 제공한다
  - 자바 9에서 리액티브 프로그래밍이라는 병렬 실행 기법 제공

### 자바 8에서 제공하는 새로운 기술
1. 스트림 API
   - 데이터베이스 질의 언어에서 표현식을 처리하는 것처럼 병렬 연산을 지원한다
   - `synchronized`를 사용하지 않아도 된다
   - 조금 다른 관점에서 보면 스트림 덕분에 메서드를 코드에 전달하고, 인터페이스에 디폴트 메서드가 존재할 수 있음을 알 수 있다
2. 메서드에 코드를 전달하는 기법
   - 스트림 API 때문에 메서드에 코드르 전달하는 기법이 생겼다고 추리하는 것은 기법의 활용성을 제한할 수도 있다.
   - 새롭고 간결한 방식으로 동작 파라미터화(Behavior Parameterization)을 구현할 수 있다
   - 약간만 다른 두 메서드를 인수를 이용해 다르게 동작하도록 하나의 메서드로 합치는 것이 바람직할 때 사용할 수 있다.
   - 함수형 프로그래밍에서 위력을 발휘한다
3. 인터페이스의 디폴트 메서드

## 왜 아직도 자바는 변화하는가?
특정 분야에서 장점을 가진 언어는 다른 경쟁 언어를 도태시킨다. 단지 새로운 하나의 기능 때문에 기존 언어를 버리고 새로운 언어와 툴 체인으로 바꾼다는 것은 쉽지않다. 하지만 새로 프로그래밍을 배우는 사람은 자연스럽게 새로운 언어를 선택하게 되며, 기존 언어는 도태된다. 자바는 첫 버전이 공개된 후 경쟁 언어를 대신하여 커다란 생태계를 성공적으로 구축했다.

### 프로그래밍 언어 생태계에서 자바의 위치
- 자바는 처음부터 유용한 라이브러리를 포함하는 잘 설계된 객체지향 언어로 시작했다
- 코드를 JVM 바이트 코드로 컴파일하는 특징(OS에 독립적) 때문에 인터넷 애플릿 프로그램의 주요 언어가 되었다
  - 일부 분야에서는 JVM 기반언어인 스칼라, 그루비가 자바를 대체했다
- 프로그래머는 빅데이터라는 도전에 직면하면서 멀티코어 컴퓨터나 컴퓨팅 클래스터를 이용해서 빅데이터를 효과적으로 처리할 필요성이 커졌다
  - 지금까지의 자바로는 불가능 했다
  - 대용량 데이터와 멀티 코어 CPU를 효과적으로 활용해야 했다
  - 자바 8은 다양한 프로그래밍 도구를 제공하고, 다양한 프로그래밍 문제를 빠르고 정확하게 그리고 쉽게 유지보수 할 수 있도록 한다.
  - 자바는 변화하는 환경에서 시장의 새로운 요구를 충족하기 위해 계속해서 발전해나간다

### 스트림 처리
- 스트림은 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임이다.
  - 어떤 프로그램의 출력 스트림은 다른 프로그램의 입력 스트림이 될 수 있다
  - 유닉스에서 파이프(|)를 사용해 스트림을 처리하는 명령을 연결할 수 있다.
    - 여러 명령을 병렬로 실행한다.
    - 즉, 한 명령이 완료되지 않아도 다음 명령이 실행된다
- 자바 8에서는 `java.util.stream` 패키지에 스트림 API가 추가되었다.
  - `Stream<T>`는 T 형식으로 구성된 일련의 항목을 의미한다.
  - 스트림 API는 파이프라인을 만드는 데 필요한 많은 메서드를 제공한다
- 스트림 API의 핵심은 기존에는 한 번에 한 항목을 처리했지만, **자바 8에서는 작업을 고수준으로 추상화 하여 일련의 스트림으로 만들어 처리할 수 있다는 것이다.**
- 스트림 파이프라인을 이용해서 여러 CPU 코어에 쉽게 할당할 수도 있다
- **스레드라는 복잡한 작업을 사용하지 않으면서 공짜로 병렬성을 얻을 수 있다.**

### 동작 파라미터화로 메서드에 코드 전달하기
- 자바 8이전에는 자바에서는 메서드를 다른 메서드로 전달할 방법이 없었다
- Comparator 객체를 sort에 넘겨주는 방법도 있지만, 이는 너무 복잡하고 기존 동작을 단순하게 재활용한다는 측면에서도 맞지 않다(?)
- 자바 8에서는 메서드를 다른 메서드의 인수로 넘겨주는 기능을 제공한다
- 이러한 기능을 이론적으로 **동작 파라미터화**라고 한다
- 동작 파라미터화가 중요한 이유는 **스트림 API는 연산의 동작을 파라미터화할 수 있는 코드를 전달한다는 사상에 기초**하기 때문이다.

### 병렬성과 공유 가변 데이터
- 스트림 메서드로 전달하는 코드는 다른 코드와 동시에 실행하더라도 안전하게 실행될 수 있어야 한다
  - 안전하게 실행할 수 있는 코드를 만드려면 **공유된 가변 데이터에 접근하지 않아야 한다**.
  - 공유된 변수나 객체가 있으면 병렬성에 문제가 발생한다.
- **공유되지 않은 가변 데이터, 메서드, 함수를 다른 메서드로 전달**하는 두 가지 기능은 함수형 프로그래밍 패러다임의 핵심적인 사항이다.

### 자바가 진화해야 하는 이유
- 언어는 하드웨어나 프로그래머 기대의 변화에 부응하는 방향으로 진화해야 한다.

## 자바 함수
- 프로그래밍 언어에서 **함수**라는 용어는 메서드 특히 **정적 메서드**와 같은 의미로 사용된다.
- 자바의 함수는 이에 더해 수학적인 함수처럼 사용되며, 부작용을 일으키지 않는 함수를 의미한다.
- 자바 8에서 함수를 새로운 값의 형식을 추가했다.
  - 멀티 코어에서 병렬 프로그래밍을 활용할 수 있는 스트림과 연계될 수 있도록 함수를 만들었기 떄문이다.
- 프로그래밍 언어에서 핵심은 값을 바꾸는 것이다.
  - 이 값을 일급 시민(First-class citizen)이라 한다
  - 전달 할 수 없는 구조체는 이급 시민이다.
  - 런타임에 메서드를 전달할 수 있다면 프로그래밍에 유용하게 사용될 수 있다.

### 메서드와 람다를 일급 시민으로
- 자바 8은 메서드를 값으로 취급할 수 있도록 설계되었다
**메서드 참조**

```java
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
    public boolean accept(File file) {
        return file.isHidden();
    }
})
```
- File 클래스에 이미 `isHidden`메서드가 있는데 굳이 FileFilter로 isHidden을 감싼 다음에 FileFilter를 인스턴스화 했다.

```java
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```
- 자바 8에서 제공하는 메서드 참조를 사용하면 위 처럼 간단하게 코드를 작성할 수 있다
- 이미 File의 isHidden이라는 함수는 준비되어 있으므로 메서드 참조를 이용해서 listFiles에 직접 전달했다

**람다: 익명 함수**
- 자바 8에서는 메서드를 일급 값으로 취급할 수 있을 뿐 아니라 람다(또는 익명 함수)를 포함하여 함수도 값으로 취급할 수 있다
- Util 클래스를 만들어 메서드를 정의해 사용할 수도 있지만, 이용할 수 있는 편리한 클래스나 메서드가 없을 때 람다 문볍을 이용하여 더 간결하게 코드를 구현할 수 있다

<img width="354" alt="image" src="https://user-images.githubusercontent.com/60502370/164883844-994fbc1f-e8b5-4dde-bade-3b6a693f94ba.png">

## 스트림
- 거의 모든 자바 애플리케이션은 컬렉션을 만들고 활용한다
  - 하지만 모든 문제가 해결되는 것은 아니다.

```java
Map<Curreny, List<Transaction>> transactionByCurrencies = new HashMap<>();

for (Transaction transaction : transactions) {
    if (transaction.getPrice() > 1000) {
        Currency currency = transaction.getCurrency();
        List<Transaction> transactionsForCurrency = transactionByCurrencies.get(currency);

        if (transactionsForCurrency == null) {
            transactionsForCurrency = new ArrayList<>();
            transactionByCurrencies.put(currency, transactionsForCurrency);
        }
        transactionsForCurrency.add(transaction);
    }
}
```

- 위 코드는 중첩된 제어 흐름 문장이 많아서 코드를 한 번에 이해하기 어렵다.
- 스트림 API를 사용하면 아래와 같이 작성할 수 있다.

```java
Map<Curreny, List<Transaction>> transactionByCurrencies = transactions.stream()
                        .filter((Transaction t) -> t.getPrice() > 1000)
                        .collect(groupingBy(Transaction::getCurrency));
```

- `for-each`를 사용한 반복을 외부 반복이라 하고, 스트림을 사용한 방법을 내부 반복이라 한다.
- 컬렉션을 사용하는 경우 거대한 리스트를 처리하기 위해 병렬로 작업하도록 할 수 있지만, 스레드를 사용하는 것은 생각만큼 쉽지 않다.

### 멀티스레딩은 어렵다
- 자바에서 제공하는 스레드 API로 멀티 스레딩 코드를 구현해서 병렬성을 이용하는 것은 쉽지 않다
- 스트림 API는 컬렉션을 처리하면서 발생하는 모호함과 반복적인 코드 문제, 그리고 멀티코어 활용이 어려움이라는 두 가지 문제를 모두 해결했다.
- 스트림 API는 데이터를 필터링, 추출, 그룹화 하는 기능을 제공한다
- 그리고 두 CPU가 존재할 때 한 CPU는 리스트의 앞 부분을 처리하도록 하고, 다른 CPU는 리스트의 뒷 부분을 처리하도록 요청할 수 있다.
  - 이러한 과정을 포킹 단계라고 한다.
- 스트림은 **스트림 내의 요소를 쉽게 병렬로 처리할 수 있는 환경을 제공한다.**

## 디폴트 메서드와 자바 모듈
- 인터페이스를 바꿔야 하는 상황에서는 인터페이스를 구현하는 모든 클래스의 구현을 바꿔야 했다.
- 자바 8에서는 인터페이스를 쉽게 변경할 수 있도록 디폴트 메서드를 제공한다.

## 함수형 프로그래밍에서 가져온 다른 유용한 아이디어
- 자바 8에서는 NPE를 피할 수 있도록 도와주는 `Optinal<T>`를 제공한다
  - `Optional<T>`는 값을 갖거나 가지지 않을 수 있는 컨테이너 객체이다
