# 리팩터링, 테스팅, 디버깅
## 가독성과 유연성을 개선하는 리팩터링
### 코드 가독성 개선
- 일반적으로 코드의 가독성이 좋다는 것은 **어떤 코드를 다른 사람도 쉽게 이해할 수 있음**을 뜻한다

### 익명 클래스를 람다 표현식으로 리팩터링 하기
```java
// 익명 클래스
Runnable r1 = new Runnable() {
    public void run() {
        System.out.println("Hello");
    }
}
// 람다
Runnable r2 = () -> System.out.println("Hello");
```

- 모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다
  - 익명 클래스에서 사용한 this, super는 람다 표현식에서 다른 의미를 갖는다
    - 익명 클래스의 this는 본인을 람다에서 this는 람다를 감싸는 클래스를 말한다
  - 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다
    - 람다는 가릴 수 없다
```java
int a = 10;
// 익명 클래스
Runnable r1 = new Runnable() {
    public void run() {
        int a = 1;
        System.out.println(a);
    }
}
// 람다
Runnable r2 = () -> {
    int a = 1; // 컴파일 에러
    System.out.println(a);
}
```

  - 익명 클래스를 람다로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다

```java
interface Task {
    public void execute():
}

public static void doSomething(Runnable r) { r.run(); }
public static void doSomething(Task t) { t.execute(); }

//익명 클래스
doSomeThing(new Task() {
    public void execute() {
        System.out.println("Hello");
    }
}) ;

// 람다
doSomeThing(() -> System.out.println("Hello"));
```
- 람다를 사용하면 같은 시그니쳐를 갖는 함수형 인터페이스를 구분할 수 없다
  - 두 가지 모두 대상 형식이 될 수 있다

```java
doSomeThing((Task)() -> System.out.println("Hello"));
```
- 위 처럼 모호함을 제거할 수 있다.
- IDE에서 자동으로 해결해준다

### 람다 표현식을 메서드 참조로 리팩터링 하기
```java
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = 
    menu.stream()
        .collect(
            groupingBy(dish -> {
                if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                return CaloricLevel.FAT;
            })
        )
```

- `getCaloricLevel()`을 Dish 클래스 내부로 구현하고 메서드 참조를 사용해 리팩토링한다

```java
public class Dish {
    ...
    public CaloricLevel getCaloricLevel() {
                if (this.getCalories() <= 400) return CaloricLevel.DIET;
                if (this.getCalories() <= 700) return CaloricLevel.NORMAL;
                return CaloricLevel.FAT;
    }
}
```

```java
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = 
    menu.stream()
        .collect(groupingBy(Dish::getCaloricLevel));
```

- comparing, maxBy 같은 정적 헬퍼 메서드를 사용하는 것도 좋다
- sum, maximum 같이 자주 사용하는 연산은 내장 헬퍼 메서드를 제공한다
  - Collectors API를 사용하면 코드의 의도가 더욱 명확히 보인다

### 명령형 데이터 처리를 스트림으로 리팩터링하기
- 외부 반복자로 데이터를 처리하는 코드는 스트림 API를 사용해 리패터링 해야한다
- 스트림 API를 사용하면 파이프라인의 의도를 더욱 명확히 보여준다

### 코드 유연성 개선
- 람다 표현식을 이용하면 동작 파라미터화를 쉽게 구현할 수 있다
  - 다양한 람다를 전달하여, 변화하는 요구사항에 대응할 수 있다

**함수형 인터페이스 적용**
- 람다 표현식을 사용하기 위해서는 함수형 인터페이스가 필요하다
- 조건부 연기 실행(Conditional deferred execution)과 실행 어라운드(Execute arround) 패턴으로 람다 리팩터링이 가능하다

**조건부 연기 실행**
```java
if (logger.isLoggable(Log.FINER)) {
    logger.finer("Problem: " + generateDiagnostic());
}
```
- 위 코드는 다음과 같은 문제가 있다
  - logger의 상태가  isLoggable이라는 메서드에 의해 클라이언트 코드로 노출된다
  - 메시지를 로깅할 때마다 logger 객체의 상태를 매번 확인해야 한다
- 메시지를 로깅하기 전에 logger 객체가 적절한 수준으로 설정되었는 지 내부적으로 확인하는 `log` 메서드를 사용하는 것이 좋다

```java
logger.log(Level.FINER, "Problem: " + generateDiagnostic());
```

- 위 코드에서는 불필요한 if문을 제거할 수 있고 logger의 상태를 노출할 필요도 없다
  - 하지만 logger가 활성화되어 있지 않더라도 메시지가 평가된다는 것이 문제다
  - 람다를 사용하면 특정 조건에서만 메시지가 생성될 수 있도록 메시지 생성 과정을 연기할 수 있어야 한다

```java
public void log(Level level, Supplier<String> msgSupplier) {
    if (logger.isLoggable()) {
        log(level, msgSupplier.get());
    }
}

// 코드
logger.log(Level.FINER, () -> "Problem: " + generateDiagnostic());
```

- `Supplier`를 통해 메시지 생성 과정을 연기할 수 있다
  - 객체 상태가 클라이언트 코드로 노출되지 않게된다

**실행 어라운드**
- 매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드가 있다면 이를 람다로 변환할 수 있다
  - 준비, 종료 과정을  처리하는 로직을 재사용한다

```java
String oneLine = processFile((BufferedReader br) -> br.readLine());
String twoLine = processFile((BufferedReader br) -> br.readLine() + br.readLine());

public static String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
        return p.process(br);
    }    
}

public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
```
  