# 함수
## 작게 만들어라
- 함수가 작을 수록 좋다는 근거를 대기는 어렵지만, 오랜 시행착오를 바탕으로 작은 함수가 좋다는 것을 확신한다
- 코드는 가로 150자를 넘어서는 안되고, 함수는 100줄을 넘어서는 안된다. 20줄도 길다
- 사실 함수는 2, 3, 4줄 정도가 적당하다

### 블록과 들여쓰기
- if 문/else 문/ while 문 등에 들어가는 블록은 한 줄이어야 한다
- 바깥을 감싸는 함수가 작아지고, 블록 안에서 호출하는 함수의 이름을 적절히 짓는다면, 코드를 이해기도 쉬워진다

## 한 가지만 해라
- 함수는 한 가지를 해야 한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다
- 함수의 한 가지 역할을 구분하기 위해서는 추상화 수준을 파악해야 한다
  - 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한 가지 작업한 한다
- 함수가 한 가지만 하는지 판단하는 다른 방법은 함수에서 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 것이다

## 함수 당  추상화 수준은 하나로
- 함수가 확실히 한 가지 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일 해야 한다
- 한 함수내 여러 추상화 수준을 섞으면 특정 표현이 근본 개념인지 아니면 세부 사항인지 뒤섞여, 읽는 사람이 헷갈리게 된다

### 내려가기 규칙
- 코드는 위에서 아래로 이야기처럼 읽혀야 좋다
- 한 함수 다음에는 추상화 수진이 한 단계 낮은 함수가 온다
  - 이를 내려가기 규칙이라 한다
- 내려가기 규칙을 따르기는 어렵지만, 이를 통해 추상화 수준을 일관되게 유지하기 쉬워진다

## Switch 문
- 본질적으로 Switch 문은 N가지 일을 처리한다
  - 불행하게도 Switch문을 완전히 피할 방법은 없다
- 다형성을 이용하여 각 switch문을 저차원 클래스에 숨기고 반복하지 않는 방법이 있다

```java
public Money calculatePay(Employee e) throws InvalidEmployeeType {
    switch(e.type) {
        case COMMISSIONED:
            return calculateCommisionedPay(e);
        case HOURLY:
            return calculateHourlyPay(e);
        case SALARIED:
            return calculateSalariedPay(e);
        default:
            throw new InvalidEmployeeType(e.type);
    }
}
```
- 위 함수의 문제점은 다음과 같다
  - 함수가 길다
  - 한 가지 작업만 수행하지 않는다
  - SRP를 위반 한다
  - OCP를 위반 한다
  - 위 함수와 구조가 동일한 함수가 무한정 존재할 수 있다
    - `isPayDay(Employee e, Date date)`, `deliverPay(Employee e, Date date)`

### Switch문 해결하기
- Switch 문을 추상 팩토리에 숨긴다
- 팩토리는 switch문을 사용해 적절한 Employee 파생 클래스의 인스턴스를 생성한다
- `calculatePay`, `isPayDay`, `deliverPay` 같은 함수는 Employee 인터페이스를 거쳐 호출된다
  - 다형성으로 인해 실제 파생 클래스에 구현된 함수가 호출된다

```java
public abstract class Employee {
    public abstract boolean isPayDay();
    public abstract Money calculatePay();
    public abstract void deliveryPay(Money pay);
}
-------------
public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}
-------------
public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
        swtich(r.type) {
            case COMMISSIONED:
                return new CommisionedEmployee(r);
            case HOURLY:
                return new HourlyEmployee(r);
            case SALARIED:
                return new SalariedEmployee(r);
            default:
                throw new InvalidEmployeeType(r.type);
        }
    }
}
```
- 다형성을 이용해서 Switch 문을 한 번만 작성하도록 한다
  - 물론 불가피한 상황도 생길 수 있다

## 서술적인 이름을 사용하라
- 서술적인 이름을 사용하는 것이 함수가 하는 일을 좀 더 잘 표현하므로 훨씬 좋은 이름이다
- 함수가 작고 단순할수록 서술적인 이름을 고르기도 쉬워진다
- 이름이 길어도 괜찮다
- 함수 이름을 정할 떄는 여러 단어가 쉽게 읽히는 명명법을 사용한다
  - 그런 다음 여러 단어를 사용해 함수 기능을 잘 표현하는 이름을 선택한다
- 서술적인 이름을 사용하면 개발자 머릿속에서도 설계가 뚜렷해지므로 코드를 개선하기 쉬워진다
- 이름을 붙일 때는 일관성이 있어야 한다
  - 모듈 내에서 함수 이름은 같은 문구, 명사, 동사를 사용한다
  - includeSetupAndTeardownPages, includeSetupPages, includeSuiteSetupPage, includeSetupPage가 좋은 예시이다


## 함수 인수
- 함수에서 가장 이상적인 인수는 무항이고 다음으로 단항, 이항, 삼항 순이다. 
  - 삼항은 피하는 편이 좋고, 4개 이상의 다항은 사용하지 않는 편이 좋다
- 인수는 어렵다
  - 함수 이름과 인수 사이에 추상화 수준이 달라질 수 있다
  - 현 시점에서 중요하지 않는 세부 사항을 알게 만든다
  - 인수가 많은 경우 테스트하기가 부담스러워진다
- 출력 인수는 입력 인수보다 이해하기 어렵다
  - 대개 함수에서 인수로 결과를 받을 것이라 기대하지 않는다
  - 출력 인수를 사용하면 코드를 다시 확인하게 만든다

### 많이 쓰는 단항 형식
- 인수에게 질문을 던지는 경우
  - `boolean fileExists("MyFile")`
- 인수를 무엇인가로 변환해 다른 결과를 반환하는 경우 
  - `InputStream fileOpen("MyFile")`
- 다소 드물지만 아주 유용한 단항 함수 형식이 이벤트이다
  - `passwordAttemptFailedNtimes(int attempts)`
  - 입력 인수만 있고 출력 인수는 없다
  - 함수가 이벤트라는 사실이 코드에 명확히 드러나야 한다

### 플래그 인수
- 플래그 인수는 추하다
- 함수가 여러가지 일을 처리하는 것을 공표하는 것과 같다
### 이항 함수
- 인수가 2개인 함수는 인수가 1개인 함수보다 이해하기 어렵다
  - `writeField(name)`이 `writeFiled(outputStream, name)`보다 이해하기 쉽다
- 이항 함수가 무조건 나쁘다는 것은 아니다
  - 불가피하게 사용하는 경우가 많다
  - 그만큼 위험이 따른다는 사실을 이해하고 가능하면 단항 함수로 바꿀 수 있도록 애써야 한다

### 삼항 함수
- 삼항 함수를 이해하는 것은 매우 어렵다
- 삼항 함수를 만들때는 신중히 고려해야 한다

### 인수 객체
- 인수가 2-3개 필요하다면 일부를 독자적인 클래스 변수로 선언할 가능성이 있다
```java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```
- x와 y를 Point로 묶었는데, 인수를 객체로 묶는 과정에서 이름을 붙여야 하므로 개념을 표현하게 된다

### 인수 목록
- 때로는 인수 개수가 가변적인 함수도 필요하다
- `String.format()`이 좋은 예이다.

### 동사와 키워드
- 함수의 의도나 인수의 순서와 의도를 제대로 표현하려면 좋은 함수 이름이 필수다
- 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다
  - `write(name)`도 좋지만, `writeField(name)`은 name이 필드라는 것을 바로 알 수 있다
- 함수 이름에 키워드를 추가할 수 있다. 즉, 함수 이름에 인수 이름을 넣는다
  - `assertEquals()`보다 `assertExpectedEqualsActual(expected, actual)`이 더 좋다

## 부수 효과를 일으키지 마라!
- 부수 효과는 거짓말이다
  - 함수에서 한 가지를 하겠다고 약속하고서 남몰래 다른 짓도 하기 때문이다
- 부수 효과는 시간적인 결합이나 순서 종속성을 초래한다

```java
public class UserValidator {
    private Cryptographer cryptographer;

    public boolean checkPassword(String userName, String password) {
        User user = UserGateway.findByName(userName);

        if (user != User.NULL) {
            String codedPhrase = user.getPhraseEncodedByPassword();
            String phase = cryptographer.decrypt(codedPhrase, password);
            if ("Valid Password".equals(phrase)) {
                Session.initialize();
                return true;
            }
        }
        return false;
    }
}
```

- `Session.initialize()`가 부수 효과를 일으킨다
  - 함수 이름만 보고서는 세션을 초기화하는 지 알 수 없다
- 이러한 부수효과가 시간적 결합을 초래한다
  - `checkPassword`는 세션을 초기화해도 괜찮은 경우에만 호출할 수 있다
- 시간적 결합이 필요하다면 `checkPasswordAndInitializeSession()`으로 이름을 바꾸는 것이 좋다
  - 한 가지 일만 한다는 규칙은 위배한다

### 출력 인수
- 일반적으로 우리는 인수를 함수의 임력으로 해석한다
- 객체 지향 프로그래밍이 나오기전에는 출력 인수를 사용하는 일이 있었지만, 이제는 사용할 필요가 없다
- 함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택한다

## 명령과 조회를 분리하라!
- 함수는 뭔가를 수행하거나, 뭔가에 답하거나 둘 중 하나만 해야 한다
- 객체 상태를 변경하거나 아니면 객체 정보를 반환하거나 둘 중 하나만 한다
```java
public boolean set(String attribute, String value);
```
- 위 함수는 attribute인 속성 앖을 찾아 value로 설정한 후 성공하면 true, 실패하면 false를 반환하도록 의도했다
```java
if (set("username", "unclebob")) {
    ...
}
```
- `username`이 `unclebob`으로 설정되어 있는지 확인하는 코드인지, `username`을 `unclebob`으로 설정하는 코드인지 명확하지 않다
- 혼란의 해결책은 명령과 조회를 분리하는 것이다
```java
if (attributeExists("username")) {
    setAttribute("username", "unclebob");
}
```
## 오류 코드보다 예외를 사용하라!
- 명령 함수에서 오류 코드를 반환하는 방식은 명령/조회 분리 규칙을 미묘하게 위반한다
- 오류 코드를 반환하면 호출자는 오류 코드를 곧바로 처리해야 한다

```java
if (deletePage(page) == E_OK) {
    if (registry.deleteReference(page.name) == E_OK) {
        if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
            logger.log("page deleted");
        } else {
            logger.log("configKey not deleted");
        }
    } else {
        logger.log("deleteReference from registry failled");
    } else {
        logger.log("delete failed");
        return E_ERROR;
    }
}
```
- 오류 코드보다 예외를 사용하면 코드가 더 깔끔해진다
```java
try {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
} catch (Exception e) {
    logger.log(e.getMessage());
}
```

### Try/Catch 블록 뽑아내기
- try/catch는 원래 추하다
  - 코드 구조에 혼란을 가져오고, 정상 옹작과 오류 처리 동작을 뒤섞는다
  - 따라서 try/catch를 따로 뽑아내는 것이 좋다

```java
public void delete(Page page) {
    try {
        deletePageAndAllReferences(page);
    } catch (Exception e) {
        logError(e);
    }
}

private void deletePageAndAllReferences(Page page) throws Exception {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) {
    logger.log(e.getMessage());
}
```

### 오류 처리도 한 가지 작업이다
- 오류 처리도 한 가지 작업이므로 오류를 처리하는 함수는 오류만 처리해야 마땅하다
- 함수에 try가 있다면 함수는 try로 시작해 catch/finally 문으로 끝나야 한다

### Error.java 의존성 자석
- 오류 코드를 반환한다는 이야기는 어디선가 오류 코드를 정의해야 한다는 의미이다
- Error.java에 변경이 생기면 Error에 의존하는 모든 클래스를 재컴파일/재배치 해야 한다
- 새 예외는 Exception 클래스에서 파생되므로 재컴파일/재배치 없이도 새 예외 클래스를 추가할 수 있다