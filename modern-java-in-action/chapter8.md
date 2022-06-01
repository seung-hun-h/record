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

## 맵 처리
- 자바 8에서는 Map 인터페이스에 몇 가지 디폴트 메서드를 추가했다

### forEach 메서드
```java
for(Map.Entry<String, Integer> entry : ageOfFriends.entrySet()) {
  String friend = entry.getKey();
  Integer age = entry.getValue();
  System.out.println(friend + " is " + age + " years old");
}
```

- 자바 8부터 Map 인터페이스는 `BiConsumer`를 인수로 받는 `forEach` 메서드를 지원한다

```java
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
```

### 정렬 메서드
- 다음 두 개의 새로운 유틸리티를 이용하면 맵의 항목 또는 키를 기준으로 정렬할 수 있다
  - Entry.comparingByValue
  - Entry.comparingByKey

```java
Map<String, String> favouriteMovies = 
              Map.ofEntries(entry("Raphael", "Star Wars"),
              entry("Cristina", "Matrix"),
              entry("Olivia", "James Bond"));

favouriteMovies.entrySet()
                .stream()
                .sorted(Entry.comparingByKey())
                .forEachOrdered(System.out::println);
```

- `forEachOrdered()`
  - Performs an action for each element of this stream, **in the encounter order of the stream if the stream has a defined encounter order.**
  - 순서 유지
  - 병렬이 아니면 `forEach()`와 차이가 없음
- `forEach()`
  - Performs an action for each element of this stream.

### HashMap의 성능
- 자바 8에서는 HashMap의 내부 구조를 바꿔 성능을 개선했다
- 기존의 맵 항목은 키로 생성한 해시코드로 접근할 수 있는 버켓에 저장했다
- 많은 키가 같은 해시코드를 반환하는 상황이 되면 O(n)의 시간이 걸리는 LinkedList로 버킷을 반환하여 성능이 저하된다
- 최근에는 버킷이 너무 커질 경우 이를 O(log(n))의 시간이 소요되는 정렬된 트리를 이용해 동적으로 치환해 충돌이 일어나는 요소 반환 성능을 개선했다
- 키가 String, Number 클래스 같은 Comparable의 형태여야만 정렬된 트리가 지원된다

### getOrDefault 메서드
- 첫 번째 인수로 키, 두 번째 인수로 맵에 키가 존재하지 않으면 반환할 기본 값을 받는다

### 계산 패턴
- 맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야 하는 상황에 사용할 수 있는 메서드가 있다
  - `computeIfAbsent`: 제공된 키에 해당하는 값이 없으면, 키를 이용해 새 값을 계산하고 맵에 추가한다
  - `computeIfPresent`: 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다
  - `compute`: 제공도니 키로 새 값을 계산하고 맵에 저장한다

- 정보를 캐시할 때 `computeIfAbsent`를 활용할 수 있다
  - 기존에 이미 데이터를 처리했다면 이 값을 다시 계산하지 않는다
```java
Map<String, byte[]> dataToHash = new HashMap<>();
MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");

line.forEach(line -> dataToHash.computeIfAbsent(line, this::calculateDigest));

private byte[] calculateDigest(String key) {
  return messageDigest.digest(key.getBytes(StandardCharsets.UTF_8));
}
```

- 여러 값을 저장하는 맵을 처리할 때도 이 패턴을 유용하게 사용할 수 있다

```java
// 기존 코드
String friend = "Raphael";
List<String> movies = frendsToMovies.get(friend);
if (movies == null) {
  movies = new ArrayList<>();
  friendsToMovies.put(friend, movies);
}

movies.add("Star Wars");

// computeIfAbsent 사용
frendsToMovies.computeIfAbsent("Raphael", name -> (new ArrayList<>()).add("Star Wars"));
```
- `computeIfPresent`는 키가 맵에 존재하고 널이 아닌 경우에 새 값을 계산한다
  - 값을 만드는 함수가 널을 반환하면 현재 매핑을 맵에서 제거한다
  - 매핑을 제거할 때는 `remove` 메서드를 오버라이드 하는 것이 적합하다

```java
Map<String, String> map = new HashMap<>();
map.put("hello", "World");

map.computeIfPresent("hello", (key, value) -> null);

System.out.println("map = " + map); // map = {}
```

### 삭제 패턴
```java
favouriteMovies.remove(key, value)
```
- 내부 구현 코드

```java
default boolean remove(Object key, Object value) {
    Object curValue = get(key);
    if (!Objects.equals(curValue, value) ||
        (curValue == null && !containsKey(key))) {
        return false;
    }
    remove(key);
    return true;
}
```

### 교체 패턴
- `replaceAll`
  - `BiFunction`을 적용한 결과로 각 항목의 값을 교체한다
- `replace`
  - 키가 존재하면 맵의 값을 바꾼다

### 합침
```java
// no merge
Map<String, String> family = Map.ofEntries(entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(entry("Raphael", "Star Wars"));

Map<String, String> everyone = new HashMap<>(family);
everyone.putAll(friends);
```
- 중복된 키가 없다면 위 코드는 잘 작동한다
- `merge`는 중복된 키를 어떻게 합칠지 결정하는 `BiFunction`을 인수로 받는다

```java
Map<String, String> family = Map.ofEntries(entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(entry("Raphael", "Star Wars"), entry("Cristina", "Matrix"));

Map<String, String> everyone = new HashMap<>(family);
everyone.forEach((k, v) -> everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
```
> 지정된 키와 연관된 값이 없거나 값이 널이면 merge는 키를 널이 아닌 값과 연결한다. 아니면 merge는 연결된 값을 주어진 매핑 합수의 결과 값으로 대치하거나 결과가 널이면 항목을 제거한다

- `merge`를 이용해 초기화 검사를 구현할 수도 있다
- 영화를 몇 회 시청했는 지 기록하는 맵이 있다고 가정

```java
// no merge
Map<String, Long> moviesToCount = new HashMap<>();
String movieName = "James Bond";
Long count = moviesToCount.get(movieName);
if (count == null) {
  moviesToCount.put(movieName, 1);
} else {
  moviesToCount.put(movieName, count + 1);
}

// merge
moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);
```

- `merge` 구현 코드
```java
default V merge(K key, V value,
    BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
                remappingFunction.apply(oldValue, value);
    if (newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```

## 개선된 ConcurrentHashMap
- 동시성 친화적이면서 최신 기술을 반영한 HashMap 버전이다
- 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다
  - `HashTable`보다 나은 성능을 보인다

### 리듀스와 검색
- `ConcurrentHashMap`은 스트림에서 봤던 것과 비슷한 세 가지 새로운 연산을 제공한다
  - `forEach`: 각 (키, 값) 쌍에 주어진 액션을 실행
  - `reduce`: 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
  - `search`: 널이 아닌 값을 반환할 때까지 (키, 값) 쌍에 함수를 적용

- 키, 값 Map.Entry, (키, 값) 인수를 이용한 네 가지 연산 형태를 제공한다
  - (키, 값): forEach, reduce, search
  - 키: forEachKey, reduceKeys, searchKeys
  - 값: forEachValue, reduceValues, searchValues
  - Map.Entry: forEachEntry, reduceEntries, searchEntries
  
- 위 연산들은 ConcurrentHashMap이 잠구지 않고 실행된다
  - 연산에 제공한 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야 한다
- 위 연산에 병렬성 기준값을 지정해야 한다
  - 맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 실행한다
  - 기준 값이 1이면 공통 스레드 풀을 이용해 병렬성을 극대화한다
  - Long.MAX_VALUE를 기준 값으로 설정하면 한 개의 스레드로 연산을 실행한다

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
long parallelismThreshold = 1;

Optional<Integer> maxValue = Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```
- 기본값에는 `reduceValuesToInt`, `reduceValuesToLong` 처럼 기본형 특화를 사용하는 것이 좋다

### 계수
- 맵의 매핑 개수를 반환하는 `mappingCount` 메서드를 제공한다
  - 기존의 size 메서드 대신 int를 반환하는 mappingCount를 사용하는 것이 좋다
  - 매핑의 개수가 int 범위를 넘어서는 이후의 상황을 대처할 수 있다

### 집합뷰
- 집합 뷰로 반환하는 `keySet` 메서드를 제공한다
- 맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받는다
- `newKeySet` 메서드를 이용해 기존의 `ConcurrentHashMap`을 유지하는 집합을 만들 수 있다

