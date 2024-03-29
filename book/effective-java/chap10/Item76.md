## 가능한 한 실패를 원자적으로 만들라
---
- 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 한다
	- 이러한 특성을 실패 원자적(failure-atomic)이라 한다

### 실패를 원자적으로 만드는 방법
1. 불변 객체를 설계하라
   - 불변 객체는 태생적으로 실패 원자적이다
   - 메서드가 실패하면 객체가 만들어지지 않기 때문이다
2. 가변 객체라면 작업 수행에 앞서 매개변수의 유효성을 검사하라
   - 객체 내부 상태를 변경하기전 잠재적인 예외 가능성을 대부분 걸러낸다
3. 실패할 가능성이 있는 모든 코드를 객체의 상태를 바꾸는 코드 보다 앞에 배치하라
   - 계산을 수행해보기 전에는 인수의 유효성을 검사해볼 수 없을 때 3번 방법에 덧붙여서 사용할 수 있다
4. 객체의 임시 복사본에서 작업을 수행한 다음 작업이 성공적으로 완료되면 원래 객체와 교채하라
   - 어떤 정렬 메서드에서는 정렬을 수행하기 전에 입력 리스트의 원소를 배열로 옮겨 담는다
   - 배열을 사용하면 정렬 알고리즘의 반복문에서 원소들에 훨씬 빠르게 접근할 수 있기 때문이다
5. 작업 도중 발생하는 실패를 가로채는 복구 코드를 작성하여 작업 전 상태로 되돌려라

- 실패 원자성은 권장되지만 항상 달성할 수 있는 것은 아니다
	- 두 스레드가 동기화 없이 객체르 동시에 수정한다면 일관성이 깨질 수 있다
	- 따라서 `ConcurrentModificationException`을 잡아냈다고 해서 그 객체가 여전히 쓸 수 있다고 가정해선 안된다
	- Error는 복구할 수 없으므로 `AssertionError`에 대해서는 실패 원자적으로 만드려는 시도조차 할 필요가 없다
- 실패 원자적으로 만들 수 있더라도 항상 그리 해야 하는 것도 아니다
	- 실패 원자성을 달성하기 위해 비용이 매우 큰 경우도 있기 때문이다
- 예외가 발생하더라도 객체의 상태는 호출 전과 똑같이 유지돼야 한다는 것이 기본 규칙이고, 이 규칙을 지키지 못하는 경우 실패 시의 객체 상태를 API에 명시해야 한다