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

## 람다로 객체지향 디자인 패턴 리팩터링하기
- 다양한 패턴을 유형별로 정리한 것이 디자인 패턴이다
  - 디자인 패턴은 공통적인 소프트웨어 문제를 설계할 때 재사용할 수 있는, 검증된 청사진을 제공한다

### 전략 패턴
- 전략 패턴은 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법이다

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/171763186-2e766488-6099-46dd-9e9d-6b98bc2b404e.png">

  - 알고리즘을 나타내는 인터페이스(Strategy 인터페이스)
  - 다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현
  - 전략 객체를 사용하는 한 개 이상의 클라이언트

```java
public interface ValidationStrategy {
  boolean execute(String s);
}

public class IsNumeric implements ValidationStrategy {
  public boolean execute(String s) {
    return s.matches("\\d+");
  }
}

public class IsAllowerCase implements ValidationStrategy {
  public boolean execute(String s) {
    return s.matches("[a=z]+");
  }
}
```

```java
public class Validator {
  private final ValidationStrategy strategy;
  public Validator(ValidationStrategy strategy) {
    this.strategy = strategy;
  } 

  public boolean validate(String s) {
    return strategy.execute(s);
  }
}
```

**람다 표현식 사용**
- `ValidationStrategy`는 함수형 인터페이스며 `Predicate<String>`과 같은 함수 디스크립터를 갖고있다

```java
Validator numericValidator = new Validator((String s) -> s.matches("[a-z]+"));

boolean b1 = numericValidator.validate("aaa");
```

### 템플릿 메서드 패턴
- 알고리즘의 개요를 제시한 다음에 알고리즘 일부를 고칠 수 있는 유연함을 제공해야 할 때 사용한다
  - 어떤 알고리즘을 사용하고 싶은데 그대로는 안되고 조금 수정해야 하는 상황에서 사용한다

```java
abstract class OnlineBanking {
  public void processCustomer(int id) {
    Customer customer = Database.getCustomerWithId(id);
    makeCustomerHappy(customer);
  }
  abstract void makeCustomerHappy(Customer customer);
}
```

**람다 표현식 사용**
```java
public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
  Customer customer = Database.getCustomerWithId(id);
  makeCustomerHappy.accept(customer);
}
```

### 옵저버 패턴
- 어떤 이벤트가 발생했을 때 한 객체가 다른 객체 리스트에 자동으로 알림을 보내야 하는 상황에 사용한다
  - GUI에 많이 사용된다

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/171766646-5819228b-53e4-4cad-a3f5-0b255f23618b.png">

```java
interface Observer {
  void notify(String tweet);
}

class NYTimes implements Observer {
  public void notify(String tweet) {
    if (tweet != null && tweet.contains("money")) {
      System.out.println("Breaking news in NY! " + tweet);
    }
  }
}

class Guardian implements Observer {
  public void notify(String tweet) {
    if (tweet != null && tweet.contains("queen")) {
      System.out.println("Yet more news from London... " + tweet);
    }
  }
}

class LeMonde implements Observer {
  public void notify(String tweet) {
    if (tweet != null && tweet.contains("wine")) {
      System.out.println("Today cheese, wine and news! " + tweet);
    }
  }
}
```

```java
interface Subject {
  void registerObserver(Observer o);
  void notifyObservers(String tweet);
}

class Feed implements Subject {
  private final List<Observer> observers = new ArrayList<>();

  public void registerObserver(Observer o) {
    this.observers.add(o);
  }

  public void notifyObservers(String tweet) {
    observers.forEach(o -> o.notify(tweet));  
  }
}
```

**람다 표현식 사용하기**
```java
Feed f = new Feed();
f.registerObserver((String tweet) -> {
  if (tweet != null && tweet.contains("money")) {
      System.out.println("Breaking news in NY! " + tweet);
    }
});
```

- 옵저버가 상태를 가지며, 여러 메서드를 정의하는 등 복잡하다면 람다 표현식보다 기존의 클래스 구현 방식을 고수하는 것이 바람직할 수 있다

### 의무 체인 패턴
- 작업 처리 객체의 체인을 만들 때 사용한다
  - 한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고, 다른 객체도 작업을 처리한 후 그 결과를 또 다른 객체로 전달하는 방식이다
- 일반적으로 다음으로 처리할 객체 정보를 유지하는 필드를 포함하는 작업 처리 추상 클래스로 의무 체인 패턴을 구성한다

```java
public abstract class ProcessingObject<T> {
  protected ProcessingObject<T> successor;
  public void setSuccessor(ProcessingObject successor) {
    this.successor = successor;
  }

  public T handle(T input) {
    T r = handleWork(input);
    if (successor != null) {
      return successor.handle(r);
    }
    return r;
  }
  abstract protected T handleWork(T input);
}
```

<img width="550" alt="image" src="https://user-images.githubusercontent.com/60502370/171768780-da74dd2d-25cc-4c47-a455-e1491937faf8.png">

- 템플릿 메서드 디자인 패턴이 적용됐다
- `handleWork` 메서드를 구현하여 다양한 종류의 작업 처리 객체를 만들 수 있다

```java
public class HeaderTextProcessing extends ProcessingObject<String> {
  public String handleWork(String text) {
    return "From Raul, Mario and Alan: " + text;
  }
}

public class SpellCheckerProcessing extends ProcessingObject<String> {
  public String handleWork(String text) {
    return text.replaceAll("labda", "lambda");
  }
}
```

**람다 표현식 사용**
```java
UnaryOperator<String> headerProcessing = (String text) -> "From Raoul, Mario and Alan: " + text;

UnaryOperator<String> spellCheckerProcessing = (String text) -> text.replaceAll("labda", "lambda");

Function<String, String> pipeline = headerProcessing.andThen(spellCheckerProcessing);

String result = pipeline.apply("...");
```

### 팩토리 패턴
- 인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 사용한다

```java
public class ProductFactory {
  public static Product createProduct(String name) {
    switch(name) {
      case " loan": return new Loan();
      case " stock": return new Stock();
      case " bond": return new Bond();
      default: throw new RuntimeException("No such product: " + name);
    }
  }
}
```

**람다 표현식 사용**
```java
Supplier<Product> loanSupplier = Loan::new;
final static Map<String, Supplier<Product>> map = new HashMap<>();

static {
  map.put("loan", Loan::new);
  map.put("stock", Stock::new);
  map.put("bond", Bond::new);
}
```

```java
public static Product createProduct(String name) {
  Supplier<Product> p = map.get(name);
  if (p != null) return p.get();
  throw new IllegalArgumentException("No such product " + name);
}
```
- createProduct에 생성자 인수를 여러개 넘겨야하는 상황에서는 Supplier로 해결할 수 없다
  - `TriFunction<T, U, V, R>`를 사용한다