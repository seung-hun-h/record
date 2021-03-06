# 제네릭
제네릭은 클래스와 인터페이스 그리고 메소드를 정의할 때 타입을 파라미터로 사용할 수 있도록 한다. 타입 파라미터는 코드 작성 시 구체적인 타입으로 대체되어 다양한 코드를 생성하도록 해준다. 제네릭의 이점은 다음과 같다

- 컴파일 시 강한 타입 체크를 할 수 있다
  - 컴파일시 잘못된 타입 사용으로인해 발생한는 문제점을 제거하기 위해 제네릭 코드에 대해 강한 타입 체크를 한다.  
- 타입 변환을 제거한다
  - 비제네릭 코드는 불필요한 타입 변환을 하기 때문에 프로그램 성능에 악영향을 미친다.
  - 제네릭을 사용하지 않으면 매번 타입 변환을 해주어야 한다.

제네릭은 타입, 멀티 타입, 메소드에 사용할 수 있고, 타입을 제한할 수도 있다.

- 제네릭 타입(class<T>, interface<T>)
\
제네릭 타입은 타입을 파라미터로 가지는 클래스와 인터페이스를 말한다. 제네릭을 사용하지 않으면 Object로 매개변수를 설정하여 모든 타입을 받을 수 있지만 그럴일은 거의 없고, 이럴 경우 매변 타입 변환을 해주어야한다.

- 멀티 타입 파라미터(class<K, V, ..>, interface<K, V, ..>)
\
제네릭 타입은 두 개이상의 멀티 타입 파라미터를 사용할 수 있다.

- 제네릭 메소드(<T, R> R method(T t))
\
제네릭 메소드는 매개 타입과 리턴 타입으로 타입 파라미터를 갖는 메소드를 말한다.

- 제한된 타입 파라미터(<T extends 최상위타입>)
\
타입 파라미터에 타입을 제한해야할 경우가 있다. 예를 들어, 수학적인 연산을 수행할 때 Byte, Integer, Long과 같은 타입만을 허가 해야 하는데 이때 `<T extends Number>`처럼 작성하면 `Number` 클래스를 상속한 타입만 파라미터로 전달될 수 있다.

- 와일드 카드 타입
\
제네릭 타입을 매개값이나 리턴 타입으로 이요할 때 구체적인 타입 대신에 와일드 카드를 다음 세 가지 형태로 사용할 수 있다.
  - 제네릭타입<?>: 타입 파라미터를 대치하는 구체적인 타입으로 모든 클래스나 인터페이스가 올 수 있다
  - 제네릭타입<? extends 상위타입>: 타입 파라미터를 대치하는 구체적인 타입으로 상위 타입이나 그 하위 타입이 올 수 있다
  - 제네릭타입<? super 하위타입>: 타입 파라미터를 대치하는 구체적인 타입으로 하위 타입이나 상위 타입이 올 수 있다.

제네릭 타입을 사용하고 싶지만 어떤 타입인지 신경쓰고 싶지 않을 때 와일드 카드 타입을 사용한다. 해당 타입에서만 제공하는 메서드를 사용한다면 제네릭을 사용하고 그렇지 않으면 와일드 카드를 사용한다
