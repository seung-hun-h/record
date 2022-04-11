## 목차
- [의미 있는 이름](#의미-있는-이름)
  - [의도를 분명히 밝혀라](#의도를-분명히-밝혀라)
    - [단순하지만 의도가 분명하지 않은 코드](#단순하지만-의도가-분명하지-않은-코드)
    - [의도가 드러난 코드](#의도가-드러난-코드)
    - [Cell을 객체로 변환](#cell을-객체로-변환)
  - [그릇된 정보를 피하라](#그릇된-정보를-피하라)
  - [의미 있게 구분하라](#의미-있게-구분하라)
  - [발음하기 쉬운 이름을 사용하라](#발음하기-쉬운-이름을-사용하라)
  - [검색하기 쉬운 이름을 사용하라](#검색하기-쉬운-이름을-사용하라)
  - [인코딩을 피하라](#인코딩을-피하라)
  - [자신의 기억력을 자랑하지 마라](#자신의-기억력을-자랑하지-마라)
  - [클래스 이름](#클래스-이름)
  - [메서드 이름](#메서드-이름)
  - [기발한 이름은 피하라](#기발한-이름은-피하라)
  - [한 개념에 한 단어를 사용하라](#한-개념에-한-단어를-사용하라)
  - [말 장난을 하지 마라](#말-장난을-하지-마라)
  - [해법 영역에서 가져온 이름을 사용하라](#해법-영역에서-가져온-이름을-사용하라)
  - [문제 영역에서 가져온 이름을 사용하라](#문제-영역에서-가져온-이름을-사용하라)
  - [의미 있는 맥락을 추가하라](#의미-있는-맥락을-추가하라)
    - [맥락이 불분명한 변수](#맥락이-불분명한-변수)
    - [맥락이 분명한 변수](#맥락이-분명한-변수)
  - [불필요한 맥락을 없애라](#불필요한-맥락을-없애라)
  - [마치면서](#마치면서)

# 의미 있는 이름
## 의도를 분명히 밝혀라
- 이름을 주의 깊게 살펴 더 나은 이름이 떠오르면 개선하라
- 변수, 함수, 클래스는 다음과 같은 질문에 답 할 수 있어야 한다
  - 변수, 함수, 클래스의 존재 이유는?
  - 수행 기능은?
  - 사용 방법은?
- 주석이 필요한 것은 의도를 분명히 담지 못했다는 의미이다

### 단순하지만 의도가 분명하지 않은 코드
```java
public List<int[]> getThem() {
    List<int[]> list1 = new ArrayList<>();
    for (int[] x : theList) {
        if (x[0] == 4) {
            list1.add(x);
        }
    }
    return list1;
}
```

- 코드가 복잡하지 않지만 의도가 분명하지 않다
- 문제는 코드의 단순성이 아니라 함축성이다
  - theList에는 무엇이 들어있는가?
  - theList에서 0번째 값이 어째서 중요한가?
  - 값 4는 무엇을 의미하는가
  - 함수가 반환하는 list1을 어떻게 사용하는가?

### 의도가 드러난 코드
- 지뢰찾기 게임을 만든다고 가정, theList는 게임판을 의미
- 배열의 0번째 값은 칸 상태, 값 4는 깃발이 꽂힌 상태를 의미
```java
public List<int[]> getFlaggedCells() {
    List<int[]> flaggedCells = new ArrayList<>();
    for (int[] cell : gameBoard) {
        if (cell[STATUS_VALUE] == 4) {
            flaggedCells.add(cell);
        }
    }
    return flaggedCells;
}
```
### Cell을 객체로 변환
```java
public List<Cell> getFlaggedCells() {
    List<Cell> flaggedCells = new ArrayList<>();
    for (Cell cell : gameBoard) {
        if (cell.isFlagged()) {
            flaggedCells.add(cell);
        }
    }
    return flaggedCells;
}
```
## 그릇된 정보를 피하라
- 프로그래머는 코드에 그릇된 단서를 남겨서는 안된다
  - 그릇된 단서는 코드의 의미를 흐린다
  - 널리 쓰이는 의미가 있는 단어를 다른 의미로 사용해도 안된다
- `accountList` 같이 컨테이너 유형을 이름에 넣지 않는 것이 바람직하다
- 서로 흡사한 이름을 사용하지 않도록 주의한다
- 유사한 개념은 유사한 표기법을 사용한다
  - 일관성이 떨어지는 표기법은 그릇된 정보다

## 의미 있게 구분하라
- 컴파일러나 인터프리터만 통과하려는 생각으로 코드를 구현하면 문제가 발생한다
- 컴파일러를 통과할지라도 연속된 숫자를 붙이거나 불용어(noise word)를 추가하는 방식은 적절하지 않다
  - 이름이 달라야 한다면 의미도 달라져야 한다
- Product 클래스가 있을 때, ProductInfo, ProductData는 의미가 불분명한 용어이다.
- a, the 같은 접두어를 사용하지 말라는 것은 아니다
  - zork라는 변수가 있다고해서 aZork, theZork를 사용해선 안된다
- 읽는 사람이 차이를 알 수 있도록 이름을 지어라

## 발음하기 쉬운 이름을 사용하라
- 발음하기 어려운 이름은 토론하기도 어렵다
- 프로그래밍은 사회적 활동이다

## 검색하기 쉬운 이름을 사용하라
- 문자 하나르 사용하는 이름과 상수는 텍스트 코드에서 쉽게 눈에 띄지 않는다
  - `MAX_CLASSES_PER_STUDENT`는 grep으로 찾기 쉽지만 `7`은 찾기 어렵다
- 이름의 길이는 범위 크기에 비례해야 한다
  - 로컬 변수는 길이가 짧을 수 있다

## 인코딩을 피하라
- 유형이나 범위 정보까지 인코딩에 넣으면 그만큼 이름을 해독하기 어려워진다
- 헝가리식 표기법
  - 변수 이름 앞에 타입을 명시하는 네이밍 방법
  - 프로그래밍 언어가 많은 타입을 지원하고, 클래스와 함수는 작아지기 때문에 사용하지 않아도 된다
  - 오히려 변수, 함수, 클래스 이름이나 타입을 바꾸거나, 읽기 어려워 진다
- 멤버 변수 접두어
  - 멤버 변수에 `m_` 접두어를 붙일 필요 없다
  - 클래스와 함수는 접두어가 필요 없을 정도로 작아야 마땅하다
- 인터페이스 클래스와 구현 클래스
  - 인터페이스에는 타입 정보를 인코딩하지 않는 것이 좋다
  - 굳이 필요하다면 구현 클래스에 인코딩하라(ex: `FactoryImpl`)

## 자신의 기억력을 자랑하지 마라
- 독자가 코드를 읽으면서 변수 이름을 자신이 아는 이름으로 변환해야 한다면 그 변수 이름은 바람직하지 못하다
- 문자 하나만 사용하는 변수 이름은 문제가 있다
  - 루프에서 반복 횟수를 세는 변수 i, j, k는 괜찮다
  - 단, 루프 범위가 아주 작고 다른 이름과 충돌하지 않아야 한다
- 전문가 프로그래머는 명료함이 최고라는 사실을 이해한다

## 클래스 이름
- 클래스 이름과 객체 이름은 명사나 명사구가 적합하다
- Customer, WikiPage, Account, AddressParser는 적합하다
- Manager, Processor, Data, Info 등과 같은 단어는 피한다
  - 의미가 모호하다

## 메서드 이름
- 동사나 동사구가 적합하다
- 접근자, 변경자, 조건자는 javaBean 표준에 따라 get, set, is를 사용한다
- 생성자를 중복정의할 때는 정적 팩토리 메서드를 사용한다
- 생성자 사용을 제한하려면 생성자를 `private`으로 선언한다

## 기발한 이름은 피하라
- 너무 기발하면 공감대가 없는 사람은 이해하지 못한다
- 특정 문화에서만 사용하는 농담은 피하는 것이 좋다

## 한 개념에 한 단어를 사용하라
- 추상적인 개념 하나에 단어 하나를 선택해 이를 고수한다
  - 똑같은 메서드를 클래스마다 fetch, retrieve, get으로 제각가 부르면 혼란스럽다
- 메서드 이름은 독자적이고 일관적이어야 한다
- 동일한 코드 기반에 controller, manager, driver를 섞어 쓰면 혼란스럽다
  - 세 단어가 근본적으로 다른점이 무엇인지 생각해야한다
- `일관성 있는 어휘는 코드를 사용할 프로그래머가 반갑게 여길 선물이다`

## 말 장난을 하지 마라
- 한 단어를 두 가지 목적으로 사용하지 마라.
  - 다른 개념에 같은 단어를 사용한다면 그것은 말 장난에 불과하다
- 같은 맥락인 경우 같은 단어를 사용한다
  - `add`라는 단어를 사용한 맥락이 어떤 값을 더한다(`+`)이면, 더하는 경우에만 사용한다
  - List에 원소를 삽입하는 경우 add를 사용하지말고 append나 insert를 사용하는 것이 좋다

## 해법 영역에서 가져온 이름을 사용하라
- 코드를 읽은 대상이 프로그래머라는 사실을 인지해야 한다
- 모든 이름을 도메인에서 가져오는 정책은 현명하지 못하다
- 프로그래머에게 전산 용어, 알고리즘, 수학 용어는 친숙할 수 있다
- 기술 개념에는 기술 이름이 가장 적합한 선택이다

## 문제 영역에서 가져온 이름을 사용하라
- 적절한 프로그래머 용어가 없다면 문제 영역에서 이름을 가져온다
- 해법 영역과 문제 영역을 잘 구분할 줄 알아야 한다
  - 문제 영역 개념과 관련이 깊은 코드라면 문제 영역에서 이름을 가져와야 한다

## 의미 있는 맥락을 추가하라
- 대다수의 이름은 스스로 그 의미가 분명하지 않다
  - 클래스, 함수 이름 공간에 넣어 맥락을 부여한다
  - 모든 방법이 실패하면 마지막 수단으로 접두어를 부여한다
  - firstName, lastName, street, houseNumber, state, city, zipcode 에서 state는 주소의 일부라는 것을 알 수 있다
  - 맥락이 명확하지 않는경우, addrFirstname 처럼 addr을 접두어로 부여할 수 있다

### 맥락이 불분명한 변수
```java
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluralModifier;
    if (count == 0) {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    } else if (count == 1) {
        number = "1";
        verb = "is";
        pluralModifier = "";
    } else {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";

    }

    String guessMessage = String.format(
        "There %s %s %s%s", verb, number, candidate, pluralModifier
    );

    print(guessMessage);
}
```

- 메서드가 길고, number, verb, pluralModifier가 guessMessage에 사용되는 것을 나중에야 알 수 있다

### 맥락이 분명한 변수
```java
public class GuessStatisticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;

    public String make(char candidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format(
            "There %s %s %s%s", verb, number, candidate, pluralModifier
        );
    }

    private void createPluralDependentMessageParts(int count) {
        if (count == 0) {
            thereAreNoLetters();
        } else if (count == 1) {
            thereIsOneLetter();
        } else {
            thereAreManyLetters(count);
        }
    }

    private void thereAreNoLetters() {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }

    
    private void thereIsOneLetter() {
        number = "1";
        verb = "is";
        pluralModifier = "";
    }

    
    private void thereAreManyLetters(int count) {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }
}
```

## 불필요한 맥락을 없애라
- 일반적으로 짧은 이름이 긴 이름보다 좋다
  - 단, 의미가 분명한 경우에 한해서이다
- 이름에 불필요한 맥락을 추가하지 않도록 주의한다
- Address는 클래스의 이름으로 적합하다
  - accountAddress, customerAddress는 Address 클래스의 인스턴스 이름으로 적합하지 않다

## 마치면서
- 좋은 이름을 선택하려면 설명 능력이 뛰어나야하고 문화적인 배경이 같아야 한다
- 좋은 이름을 선택하는 것은 교육 문제이다
