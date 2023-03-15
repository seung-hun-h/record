## 필요 없는 검사 예외 사용은 피하라
---
- 검사 예외는 발생한 문제를 프로그래머가 처리하여 안정성을 높이게끔 해준다
- 하지만 과하면 오히려 사용자에게 부담을 준다
	- 호출 코드에서 반드시 catch 하거나 더 바깥으로 던지게 한다
	- 스트림 안에서 사용할 수 없다
- API를 제대로 사용해도 발생할 수 있는 예외이거나, 프로그래머가 의미있는 조치를 취할 수 있다면, 그 부담은 충분히 받아들일 수 있을 것이다
- 위 두 경우에 해당하지 않는다면 비검사 예외를 사용하자

### 검사 예외를 피하는 방법
- 검사 예외를 던지는 대신 옵셔널을 반환하자
	- 검사 예외를 던지지 말고 빈 옵셔널을 반환한다
	- 예외가 발생한 이유를 알려주는 부가정보를 할 수 없다는 단점이 있다
- 검사 예외를 던지는 메서드를 2개로 쪼개서 비검사 예외로 바꿀 수 있다