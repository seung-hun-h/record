## 전통적인 for문 보다는 for-each문을 사용하라
---

```Java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) { 
	...
}

for (int i = 0; i < c.size(); i++) {
	...
}
```
- 전통적인 반복자를 사용하면 코드가 더러워지고, 버그가 숨어들 수 있다
- for-each문을 사용하면 전통적인 반복자를 사용했을 때 문제점을 해결할 수 있다

```Java
for(Element e : elemtns) {
	...
}
```

- for-each문을 사용하면 컨테이너가 배열인지 리스트인지 상관하지 않고 일관적으로 순회할 수 있다

### for-each문을 사용할 수 없는 경우
- 파괴적인 필터링
	- 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 `remove` 메서드를 호출해야 한다
	- 자바 8부터 `Collection`의 `removeIf` 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다
- 변형
	- 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 반복자나 배열의 인덱스를 사용해야 한다
- 병렬 반복
	- 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다(?)
		- 코드 58-4는 오히려 for-each문을 사용해야 하는 예제가 아닌가
	- 일단 병렬 반복이라는 뜻은 여러 컬렉션을 순회하는 것 같다
	- 만약 이런 경우라면 반복자나 인덱스 접근을 사용해야 할 듯

```Java
List<String> names = List.of("함승훈", "한승훈", "항승훈");
List<Integer> ages = List.of(28, 29, 30);
List<Person> people = new ArrayList<>();

// for-each를 사용하면 어쩔 수 없이 모든 경우의 수가 나온다
for (String name : names) {
	for(Integer age : ages) {
		people.add(new Person(name, age))
	}
}

// 반복자나 인덱스를 사용하면 값을 제대로 매핑할 수 있다
for(Iterator<String> i = names.iterator(), Iterator<Integer> j = ages.iterator(); 
 i.hasNext() && j.hasNext(); ) {
	 people.add(new Person(i.next(), j.next()));
 }
```

### for-each를 사용하기 위해서는 Iterable을 구현해야 한다
- for-each문은 컬렉션과 배열은 물론 `Iterable` 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다
- `Iterable`을 처음부터 구현하는 것은 어렵지만, 원소들의 묶음을 표현하는 타입을 작성해야 한다면 `Iterable`을 구현하는 것을 생각해 보아야 한다