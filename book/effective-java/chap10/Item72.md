## 표준 예외를 사용하라
---
- 예외는 다른 코드와 마찬가지로 재사용하는 것이 좋다
	- 자바 라이브러리는 대부분 API에서 쓰기에 충분한 수의 예외를 제공한다

### 표준 예외의 장점
1. 우리가 설계한 API가 다른 사람이 익히고 사용하기 쉬워진다
2. 우리의 API를 사용한 프로그램도 낯선 예외를 사용하지 않으므로 읽기 쉽게 된다
3. 예외 클래스 수가 적을수록 메모리 사용량도 줄어들고 클래스를 적재하는 시간도 적게 걸린다

### 표준 예외들
| 예외                            | 주요 쓰임                                                               |
| ------------------------------- | ----------------------------------------------------------------------- |
| IllegalArgumentException        | 허용하지 않는 값이 인수로 건네졌을 때(null은 따로 NullPointerException) |
| IllegalStateException           | 객체가 메서드를 수행하기에 적절하지 않은 상태일 때                      |
| NullPointerException            | null을 허용하지 않는 메서드에 null을 건넸을 때                          |
| IndexOutOfBoundsException       | 인덱스가 범위를 넘었을 때                                               |
| ConcurrentModificationException | 허용하지 않는 동시 수정이 발견됐을 때                                   |
| UnsupportedOperationException   | 호출한 메서드를 지원하지 않을 때                                        |

- Exception, RuntimeExcetion, Throwable, Error은 직접 재사용하지 말자. 추상체라고 생각하자

### 표준 예외를 선택하기 어려울 떄
- 표준 예외의 쓰임이 상호 배타적이지 않아 헷갈릴 때가 있다
- 대표적으로 IllegalArgumentException, IllegalStateException의 쓰임을 구분하는 경우가 있다
	- 인수로 건넨 수 만큼 카드를 뽑아 나눠주는 메서드를 구현한다고 가정한다
	- 덱에 남아있는 카드 수 보다 큰 수를 인자로 건네면 어떤 예외를 던저야 할까
	- 인수의 값이 너무 크다고 보면 IllegalArgumentException, 카드 수가 너무 적다고 보면 IllegalStateException이다
	- 인수의 값이 무엇이었든 어차피 실패했을 거라면 IllegalStateException을 사용하고, 그렇지 않으면 IllegalArgumentException을 사용하자
