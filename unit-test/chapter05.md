# 5. 목과 테스트 취약성
- 목을 사용하는 것은 논란의 여지가 있는 주제이다
- 목을 사용하면 리팩터링 내성이 부족한 테스트를 초래할 수 있다
- 하지만 목을 적용할 수 있는 경우가 있고, 목 사용이 바람직한 경우도 있다

## 5.1 목과 스텁 구분
### 5.1.1 테스트 대역 유형
- 테스트 대역은 모든 유형의 비운영용 가짜 의존성을 설명하는 포괄적인 용어이다
- 테스트 대역에는 더미, 스텁, 스파이, 목, 페이크 다섯 가지가 있다
- 목: 외부로 나가는 상호 작용을 모방하고 검사하는데 도옴이 된다. 이러한 상호 작용은 SUT가 상태를 변경하기 위한 의존성을 호출하는 것에 해당한다
  - 목: 목 프레임워크의 도움을 받아 생성
  - 스파이: 수동으로 작성
- 스텁: 내부로 들어오는 상호 작용을 모방하는데 도움이 된다. 이러한 상호 작용은 SUT가 입력 데이터를 얻기 위한 의존성을 호출하는 것에 해당한다
  - 스텁: 시나리오마다 다른 값을 반환할 수 있게끔 구성할 수 있도록 필요한 것을 다 갖춘 완전한 의존성
  - 더미: 널 값이나 가짜 문자열과 같이 단순하고 하드 코딩된 값
  - 페이크: 스텁과 비슷하다. 보통 아직 존재하지 않는 의존성을 대체하고자 구현한다

- 목은 상호 작용을 모방하고 검사하는 반면 스텁은 모방만 한다는 차이점은 중요하다

### 5.1.2 도구로서의 목과 테스트 대역으로서의 목
- 목은 테스트 대역의 의미뿐 아니라 목 라이브러리의 클래스도 의미한다
- 태스트 대역으로서의 목과 도구로서의 목이 존재하는 것이다

```java
void test1() {
    IEmailGateway mock = Mockito.mock(IEmailGateway.class);
    Controller sut = new Controller(mock);

    sut.greetUser("user@gmail.com");
    Mockito.verify(mock, times(1)).sendGreetingsEmail("user@gmail.com"); 
}
```

- 도구로서의 목을 이용해 목과 스텁 두 가지 유형의 테스트 대역을 생성할 수 있기 때문에 도구로서의 목과 테스트 대역으로서의 목을 혼동하지 않는 것이 중요하다
- 다음 예제 테스트에서도 목 클래스를 사용하지만 해당 클래스의 인스턴슨는 목이 아니라 스텁이다

```java
void test2() {
    IDatabse stub = Mockito.mock(IDatabse.class);
    Mockito.when(stub.getNumberOfUsers()).thenReturn(10);
    Controller sut = new Controller(mock);

    Report report = sut.createReport();

    Assertions.assertThat(report.getNumberOfUsers()).isEqualTo(10);
}
```

- 이 테스트 대역은 SUT에 입력 데이터를 제공하는 호출을 모방한다
- test1의 `sendGreetingEmail()`에 대한 호출은 외부로 나가는 상호 작용이고 그 목적은 부작용을 일으키는 것뿐이다

### 5.1.3 스텁으로 상호 작용을 검증하지 마라
- 목은 SUT에서 관련 의존성으로 나가는 상호 작용을 모방하고 검사한다
- 스텁은 내부로 들어오는 상호 작용만 모방하고 검사하지 않는다
  - 이러한 호출은 최종 결과를 산출하기 위한 수단일 뿐이다
- 테스트에서 거짓 양성을 피랗고 리팩터링 내성을 향상시키는 방법은 구현 세부 사항이 아니라 최종 결과를 검증하는 것 뿐이다
- 결과가 올바르다면 SUT가 최종 결과를 어떻게 생성하는지는 중요하지 않다
- 다음 예제는 스텁으로 상호 작용을 검증하여 꺠지기 쉬운 테스트이다

```java
void test2() {
    IDatabse stub = Mockito.mock(IDatabse.class);
    Mockito.when(stub.getNumberOfUsers()).thenReturn(10);
    Controller sut = new Controller(mock);

    Report report = sut.createReport();

    Assertions.assertThat(report.getNumberOfUsers()).isEqualTo(10);
    Mock.verify(stub, times(1)).getNumberOfUsers();
}
```

- 최종 결과가 아닌 사항을 검증하는 이러한 관행을 과잉 명세(overspecification)이라 한다

### 5.1.4 목과 스텁 함께 쓰기
- 목과 스텁의 특성을 모두 나타내는 테스트 대역을 만들 필요가 때로는 있다

```java
public void test() {
    IStore storeMock = Mockito.mock(IStore.class);
    Mockito.when(storeMock.hasEnoughInventory(Product.shampoo, 5)).thenReturn(false);
    Customer sut = new Customer();

    boolean success = sut.purchase(storeMock, Product.shampoo, 5);

    Assertions.assertThat(success).isFalse();
    Mockito.verify(storeMock, times(0)).removeInventory(Product.shampoo, 5);
}
```

### 5.1.5 목과 스텁은 명령과 조회에 어떻게 관련돼 있는가?
- 목과 스텁의 개념은 명령 조회 분리(CQS, Command Query Seperation) 원칙과 관련있다
- CQS 원칙에 따르면 모든 메서드는 명령이거나 조회여야 하며, 둘을 혼용해서는 안된다
- 명령은 부작용을 일으키고 어떤 값도 반환하지 않는 메서드다
  - 부작용을 일으키는 메서드는 항상 동일한 결과를 반환해야 한다. 즉, 멱등성을 보장해야 한다
- 조회는 부작용이 없고 값을 반환한다
- 항상 CQS 원칙을 따를 수는 없지만 최대한 따르는 것이 좋다