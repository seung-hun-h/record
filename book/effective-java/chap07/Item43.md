## 람다 보다는 메서드 참조를 사용하라
---
- 메서드 참조를 사용하면 람다 보다도 간결하게 코드를 작성할 수 있다

```Java
map.merge(key, 1, (count, incr) -> count + incr);
```

- Map의 merge는 키, 값, 함수를 인자로 받는다
- 주어진 키가 맵 안에 아직 없다면 주어진 (키, 값) 쌍을 그대로 저장한다
- 키가 이미 있다면 함수를 현재 값과 주어진 값에 적용한 다음 그 결과로 현재 값을 덮어쓴다
- 즉 맵에 (키, 함수의 결과) 쌍을 저장한다

- 이에 메서드 참조를 사용하면 아래와 같이 작성할 수 있다

```Java
map.merge(key, 1, Integer::sum);
```
- 람다로 할 수 없는 일이라면 메서드 참조로도  할 수 없다
- 그렇더라도 메서드 참조를 사용하는 편이 보통은 더 짧고 간결하므로, 람다로 구현했을 때 너무 길거나 복잡하다면 메서드 참조가 좋은 대안이 되어준다

### 메서드 참조를 사용하는 것이 더 안좋은 경우
- 때로는 람다가 메서드 참조보다 더 간결한 경우가 있다

```Java
service.execute(GoshThisClassNameIsHumongous::action);
service.execute(() -> action());
```

```Java
Function.identity();
x -> x
```

- 두 가지 경우 모두 람다가 더 간결하다

### 메서드 참조의 유형

| 메서드 참조 유형   | 예                     | 같은 기능을 하는 람다                                 |
| ------------------ | ---------------------- | ----------------------------------------------------- |
| 정적               | Integer::parseInt      | str -> Integer.parseInt(str)                          |
| 한정적(인스턴스)   | Instant.now()::isAfter | Instant then = Instant.now()     t -> then.isAfter(t) |
| 비한정적(인스턴스) | String::toLowerCase    | str -> str.toLowerCase()                              |
| 클래스 생성자      | TreeMap<K,V>::new      | () -> new TreeMap<K, V>                               |
| 배열 생성자        | int[]::new             | len -> new int[len]                                   |

