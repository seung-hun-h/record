## 스트림에서는 부작용 없는 함수를 사용하라
---
- 스트림은 함수형 프로그래밍에 기초한 패러다임이다
	- 스트림이 제공하는 표현력, 속도, 병렬성을 얻으려면 이 패러다임을 받아드려야 한다
- 스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다
- 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다
	- 순수 함수란 오직 입력만이 결과에 영향을 주는 함수를 말한다
	- 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다

**스트림 패러다임을 이해하지 못한 코드**
```Java
Map<String, Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()) {
	words.forEach(word -> {
		freq.merge(word.toLowerCase(), 1L, Long::sum);
	});
}
```
- 위 코드는 스트림 코드를 가장한 반복적 코드다
	- 읽기 어렵고 유지 보수에도 좋지 않다

**스트림을 제대로 활용한 코드**
```Java
Map<String, Long> freq;
try(Stream<String> words = new Scanner(file).tokens()) {
	freq = words
			.collect(groupingBy(String::toLowerCase, counting()));
}
```

- `forEach`연산은 종단 연산 중 기능이 가장 적고 가장 '덜' 스트림답다
- `forEach` 연산은 스트림 계산 결과를ㄹ 보고할 때만 사용하고, 계산하는 데는 쓰지 말자
---
Collectors 관련한 내용은 생략
