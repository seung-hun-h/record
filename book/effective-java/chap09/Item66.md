## 네이티브 메서드는 신중히 사용하라
---
- 자바 네이티브 인터페이스는 자바 프로그램의 네이티브 메서드를 호출하는 기술이다
- 네이티브 메서드는 C나 C++같은 네이티브 프로그래밍 언어로 작성된 메서드를 말한다

### 네이티브 메서드의 쓰임새
- 레지스트리 같은 플랫폼 특화 기능 사용
- 네이티브 코드로 작성된 기존 라이브러리 사용
- 성능 개선의 목적
	- 자바의 BigInteger는 자바 8에서 곱셈 성능을 개선한 이후 개선되지 않는다
	- 네이티브 라이브러리는 GNU 다중 정밀 연산 라이브러리를 개선해왔다
	- 고성능 정밀 연산이 필요한 프로그래머라면 네이티브 메서드를 사용하는 것을 고려해도 좋다

### 네이티브 메서드의 단점
- 성능을 개선할 목적으로 네이티브 메서드를 사용하는 것은 권장하지 않는다
- 네이티브 프로그래밍 언어가 안전하지 않으므로, 네이트브 메서드를 사용하는 애플리케이션도 안전하지 않다
- 이식성도 낮고, 디버깅도 어렵고, 주의하지 않으면 성능이 오히려 하락한다
- 네이티브 메서드와 자바 코드 사이에 접착 코드도 작성해야 하는데 귀찮다

