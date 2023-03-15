## 복구할 수 있는 상황일때는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라
---
- 호출하는 쪽에서 복구하리라 여겨지는 상황이라면 검사 예외를 사용하라
- API 설계자는 API 사용자에게 검사 예외를 주어 그 상황에서 회복해내라고 요구할 수 있다
	- 예외를 잡기만하고 아무 것도 안할 수 있는데, 보통 좋지 않은 생각이다
- 비검사 throwable은 런타임 예외와 에러다
	- 프로그램에서 잡을 필요가 없거나 통상적으로는 잡지 말아야 한다
- 프로그래밍 오류를 나타낼 때는 런타임 예외를 사용하자
	- 런타임 예외의 대부분은 전제조건을 만족하지 못했을 때 발생한다
- Error 클래스를 상속하지 말자
	- 자바 언어 명세가 요구한 것은 아니지만 널리퍼진 규약이다
	- 우리가 구현하는 모든 비검사 예외는 RuntimeException의 하위 클래스여야 한다

### 예외도 객체다
- 예외 역시 어떤 메서드라도 정의할 수 있는 완벽한 객체다
- 예외의 메서드는 주로 그 예외를 일으킨 상황에 관한 정보를 코드 형태로 전달하는데 쓰인다
	- 이런 메서드가 없으면 문자열을 파싱하기도 하는데 이는 나쁜 습관이다
	- 오류 메시지 포맷을 상세히 기술하지 않으므로, 버전이 변경되면서 오류 메시지 또한 변경될 수 있다
