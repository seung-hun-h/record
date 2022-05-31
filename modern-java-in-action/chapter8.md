# 컬렉션 API 개선
## 컬렉션 팩토리
```java
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
```
- `Arrays.asList` 팩토리 메서드를 사용하면 간단하게 코드를 작성할 수 있다
- 하지만 새 요소를 추가하거나 요소를 삭제하면 `UnsupportedOperationException`이 발생한다

**UnsupportedOperationException 발생**
- 내부적으로 고정된 크기의 변환할 수 있는 배열로 구현되어있기 때문에 예외가 발생한다
  - `java.util.Arrays`에 고정된 배열로 `ArrayList`가 구현되어 있다

- 하지만 Set, Map을 만들 수 있는 `asSet`, `asMap`과 같은 팩토리 메서드는 존재하지 않는다
- 자바 9에서는 List, Set, Map을 쉽게 만들 수 있는 팩토리 메서드를 제공한다

### 리스트 팩토리
```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");

friends.add("Chih-Chun");
```

- 위 코드를 실행하면 `java.lang.UnsupportedOperationException`이 발생한다
  - 변경할 수 없는 리스트가 만들어졌기 때문이다.(`ImmutableCollections`을 사용한다)
- 변경할 수 없는 제약이 나쁜 것은 아니다
  - 컬렉션이 의도치 않게 변하는 것을 막을 수 있기 때문이다
  - 하지만 요소 자체가 변하는 것은 막을 수 없다
  - 리스트를 바꿔야 하는 상황이면 직접 리스트를 만들면 된다
  - NULL을 금지하므로 의도치 않은 버그를 방지할 수 있다

- 데이터 처리 형식을 설정하거나 데이터를 변환할 필요가 없다면 사용하기 간편한 팩토리 메서드를 이요할 것을 권장한다

### 오버로딩 VS 가변 인수
- List의 인터페이스를 살펴보면 `List.of`의 다양한 오버로딩 버전이 있다는 것을 알 수 있다

```java
static <E> List<E> of(E e1, E e2, E e3, E e4) {
    return ImmutableCollections.listFromTrustedArray(e1, e2, e3, e4);
}
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5) {
    return ImmutableCollections.listFromTrustedArray(e1, e2, e3, e4, e5);
}
```

- 가변 인수`List.of(E... elements)`를 사용하지 않은 버전이 존재한다
- 가변 인수 버전은 추가 배열을 할당해서 리스트로 감싼다
  - 배열을 할당하고 초기화하며 나중에 가비지 컬렉션을 하는 비용을 지불해야 한다
- 고정된 숫자의 요소를 API로 정의하므로 가벼 인수를 사용하면서 발생하는 비용을 제거할 수 있다

### 집합 팩토리
```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");

// IllegalArgumentException
Set<String> friends = Set.of("Raphael", "Olivia", "Olivia");
```
- 중복된 요소가 존재하면 `IllegalArgumentException`이 발생한다
### 맵 팩토리
- `Map.of` 팩토리 메서드는 키와 값을 번갈아 작성한다
- 열 개 이하의 키와 값 쌍을 가진 맵을 만들때 유용하다

```java
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
```
- `Map.ofEntries`는 `Map.Entry<K, V>` 객체를 인수로 받으며 가변 인수로 구현되어있다
- 열 개 이상의 키와 값 쌍을 가진 맵을 만들때 사용하는 것이 좋다

```java
import static java.util.Map.entry;

Map<String, Integer> ageOfFriends = Map.of(entry("Raphael", 30), entry("Olivia", 25), entry("Thibaut", 26));
```

## 리스트와 집합 처리
- 자바 8에서는 List, Set 인터페이스에 다음과 같은 메서드를 추가했다
  - `removeIf`: 프레디케이트를 만족하는 요소를 제거한다
  - `replaceAll`: List 인터페이스에서 이용할 수 있는 기능으로 UnaryOperator 함수를 이용해 요소를 바꾼다
  - `sort`: List 인터페이스에서 제공하는 기능으로 리스트를 정렬한다
- 컬렉션을 바꾸는 동작은 에러를 유발하며 복잡합을 더하기 때문에 위 메서드처럼 기존 컬렉션 자체를 바꾸는 메서드가 나타났다

### removeIf 메서드
```java
for (Transaction transaction : transactions) {
  if (Character.isDigit(transaction.getReferenceCode()).charAt(0)) {
    transaction.remove(transaction);
  }
}
```

- 위 코드는 `ConcurrentModificationException` 발생
  - for-each는 내부적으로 `Iterator`를 사용하기 때문이다

```java
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext(); ) {
  Transaction transaction = iterator.next();
  if (Character.isDigit(transaction.getReferenceCode()).charAt(0)) {
    transaction.remove(transaction);
  }
}
```
- 위 코드에서 반복자와 컬렉션의 상태가 서로 동기화되지 않는다
  - `Iterator` 객체는 `next()`, `hasNext()`를 이용해 소스를 질의 한다
  - `Collection` 객체는 `remove()`를 호출해 요소를 삭제한다

```java
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext(); ) {
  Transaction transaction = iterator.next();
  if (Character.isDigit(transaction.getReferenceCode()).charAt(0)) {
    iterator.remove();
  }
}
```
- 반복자의 `remove()`를 호출해 위 문제를 해결했다
- 위 패턴을 자바 8의 `removeIf`로 바꿀 수 있다

```java
transactions.removeIf(transaction -> Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

### replaceAll 메서드
- `List` 인터페이스의 `replaceAll` 메서드를 이용해 리스트의 각 요소를 새로운 요소로 바꿀 수 있다
- 스트림 API를 사용하면 다음처럼 문제를 해결할 수 있었다


```java
referenceCodes.stream()
          .map(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1))
          .collect(Collectors.toList())
          .forEach(System.out::println);
```

- 위 코드는 새로운 컬렉션을 만든다. 목표는 기존의 컬렉션을 바꾸는 것이다

```java
for (ListIterator<String> iterator = referenceCodes.listIterator(); iterator.hasNext(); ) {
  String code = iterator.next();
  iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
}
```

- 반복자의 `set()`을 호출해서 컬렉션을 변경할 수 있다
- 위 코드는 자바 8의 List 인터페이스에서 제공하는 `replaceAll`로 바꿀 수 있다

```java
refereceCodes.replaceAll(code -> code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```

