## 추상화 수준에 맞는 예외를 던져라
---
- 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다
	- 이를 예외 번역(Exception Translation)이라 한다

```Java
try {
	...
} catch (LowerLevelException exception) {
	throw new HigherLevelException(...);
}
```

- 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄를 사용하는 것이 좋다
```Java
try {
	...
} catch (LowerLevelException exception) {
	throw new HigherLevelException(exception);
}
```
- 대부분의 표준 예외는 예외 연쇄용 생성자를 갖추고 있다
- 그렇지 않은 예외라도 Throwable의 initCause 메서드를 이용해 원인을 직접 못밖을 수 있다

- 무턱대고 예외를 전파하는 것 보다는 예외 번역이 우수한 방법이지만, 그렇다고 남용하면 안된다
	- 가능하면 저수준 메서드가 반드시 성공하도록하여 아럐 계층에서는 예외가 발생하지 않도록하는 것이 최선이다
	- 때로는 상위 계층 메서드의 매개변수 값을 아래 계층 메서드로 건네기 전에 미리 검사하는 방법으로 이 목적을 달성할 수 있다
	- 차선책으로, 아래 계층에서 예외를 피할 수 없다면, 상위 계층에서 그 예외를 조용히 처리하여 문제를 API 호출자에까지 전파하지 않는 방법이 있다
		- 발생한 예외를 로깅하여, 프로그래머가 로그를 분석해 추가 조치를 취할 수 있도록 할 수 있다

