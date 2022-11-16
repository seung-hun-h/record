## 스트림 병렬화는 주의해서 사용하라
---
- 자바 8부터는 `parallel` 메서드만 한 번 호출하면 파이프라인을 병렬 실행할 수 있는 스트림을 지원한다
	- 자바로 동시성 프로그램을 작성하기가 점점 쉬워지고 있지만, 올바르고 빠르게 작성하는 일은 여전히 어렵다
- 동시성 프로그래밍을 할 떄는 안전성과 응답 가능 상태를 유지하기 위해 애써야 하는데, 병렬 스트림 파이프라인 프로그래밍에서도 다를게 없다

```Java
public static void main(String[] args) {
	primes().map(p -> TWO.pow(p.intValueExact()).substract(ONE))
			.filter(mersenne -> mersenne.isProbablePrime(50))
			.limit(20)
			.forEach(System.out::println);
}

static Stream<BigInteger> primes() {
	return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```

- 위 코드의 성능을 높이기 위해서 `parallel`을 호출하면 제대로 동작하지 않는다
- 환경이 아무리 좋더라도 `Stream.iterate`거나 중간 연산으로 `limit`를 사용한다면 파이프라인 병렬화로 성능 개선을 기대할 수 없다
- 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스이거나, int 범위, long 범위일 때 병렬화의 효과가 가장 좋다
	- 이들 자료구조는 모두 원하는 크기로 손쉽게 나눌 수 있어서 일을 다수의 스레드에 분배하기 좋다는 특징이 있다
	- 그리고 이들 자료구조는 참조 지역성이 좋다. 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 뜻이다
- 병렬화에 가장 좋은 연산은 축소(reduction)이다
	- 종단 연ㅅ난에서 수행하는 작업량이 파이프라인 전체 작업에서 상당 비중을 차지하면서 순차적인 연산이라면 파이프라인 병렬 수행의 효과는 제한된다
	- min, max,  count, sum 같이 완성된 형태로 제공하는 연산
	- anyMaych, allMatch, noneMatch 같이 조건에 맞으면 바로 반환되는 메서드도 병렬에 적합하다

### 병렬화를 잘못하면
- 스트림을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다
	- 스트림을 병렬화하려면 강도 높게 테스트 해야 한다