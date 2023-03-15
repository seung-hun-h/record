## 예외는 진짜 예외 상황에만 사용하라
---
- 예외는 오직 예외 상황에만 사용해야 한다
	- 절대로 일상적인 제어 흐름용으로 사용하면 안된다
- 표준적이고 쉽게 이해되는 관용구를 사용하고, 성능 개선을 목적으로 과하게 머리를 쓴 기법은 자제하라
	- 성능이 좋아지더라도 JVM은 계속 발전하므로, 그 우위가 오래가지 않을 수 있다

### API를 설계할 때
- 잘 설계된 API라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야 한다
- 특정 상태에서만 호출할 수 있는 '상태 의존적' 메서드를 제공하는 클래스는 '상태 검사' 메서드도 함께 제공해야 한다
	- `Iterator` 인터페이스의 `next()`, `hasNext()`는 각각 상태 의존적 메서드와 상태 검사 메서드에 해당한다
- 상태 검사 메서드 대신 옵셔널이나 null 같은 특수한 값을 반환할 수도 있다

### 상태 검사 메서드, 옵셔널, 특정 값 중 하나를 선택하는 지침
1. 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인으로 상태가 변할 수 있다면 옵셔널이나 특정 값을 사용한다. 상태 검사 메서드와 상태 의존적 메서드 호출 사이에 객체의 상태가 변할 수 있기 때문이다
2. 성능이 중요한 상황에서 상태 검사 메서드가 상태 의존적 메서드의 작업 일부를 중복 수행하고 있다면 옵셔널이나 특정 값을 선택한다
3. 다른 모든 경우엔 상태 검사 메서드 방식이 조금 더 낫다고 할 수 있다. 가독성이 살짝 더 좋고, 잘못 사용했을 떄는 발견하기가 쉽다. 상태 검사 메서드 호출을 깜빡했다면 상태 의존적 메서드가 예외를 던져 버그를 확실히 드러낼 것이다. 반면 특정 값은 검사하지 않고 지나쳐도 발견하기가 어렵다.
