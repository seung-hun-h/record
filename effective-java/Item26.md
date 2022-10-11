## ITEM26 로 타입은 사용하지 말라

### 용어 정리
| 한글 용어                | 영문 용어               | 예                                 |
| ------------------------ | ----------------------- | ---------------------------------- |
| 매개변수화 타입          | parameterized type      | `List<String>`                     |
| 실제 타입 매개변수       | actual type parameter   | `String`                           |
| 제네릭 타입              | generic type            | `List<E>`                          |
| 정규 타입 매개변수       | formal type parameter   | `E`                                |
| 비한정적 와일드카드 타입 | unbounded wildcard type | `List<?>`                          |
| 로 타입                  | raw type                | `List`                             |
| 한정적 타입 매개변수     | bounded type parameter  | `<E extends Number>`               |
| 재귀적 타입 한정         | recursive type bound    | `List<T extends Comparable<T>>`    |
| 한정적 와일드카드 타입   | bounded wildcart type   | `List<? extends Number`            |
| 제네릭 메서드            | generic method          | `static <E> List<E> asList(E[] a)` |
| 타입 토큰                | type token              | `String.class`                       |

### 제네릭 타입이란
- 제네릭 클래스, 제네릭 인터페이스는 클래스와 인터페이스 선언에 타입 매개변수가 쓰이는 것을 의미한다
- 제네릭 클래스, 제네릭 인터페이스를 통틀어 **제네릭 타입**이라 한다
- 제네릭 타입은 일련의 매개변수화 타입을 정의한다
	- `List<String>`은 원소의 타입이 String인 리스트를 뜻하는 매개변수화 타입이다
- 제네릭  타입을 하나 정의하면 그에 딸린 로 타입도 함께 정의된다
	- 로 타입은 타입 선언에서 제네릭 타입 정보가 모두 지워진 것처럼 동작한다

### 제네릭 타입을 사용하는 이유
- 제네릭은 타입 안전성과 표현력을 제공한다
	- 로 타입을 사용하면 제네릭이 제공하는 기능을 사용할 수 없게 된다
	- 로 타입을 사용하면 런타임에야 오류를 알아 차릴 수 있다
	- 런타임에 문제를 겪는 코드는 원인과 물리적으로 떨어져 있을 가능성이 있다

```Java
Collection stamps = ...; // Stamp만 저장하기로 약속
stamps.add(new Coin()); // 실수로 Coin을 넣었다

...

for(Iterator i = stamps.iterator(); i.hasNext(); ) {
	Stamp stamp = (Stamp)i.next(); // ClassCastException을 던진다
	stamp.cancel();
}
```

- 그럼에도 로 타입을 제공하는 이유는 하위 호환성 때문이다


### 로 타입을 사용하지 말자
- `List` 같은 로 타입 보다는 `List<Object>` 처럼 임의 객체를 허용하는 매개변수화 타입을 사용하자
	- 매개변수로 `List`를 받는 메서드에 `List<String>`을 넘길 수 있지만, `List<Object>`를 받는 메서드에는 넘길 수 없다
	- `List<String>`은 `List`의 하위 타입이지만, `List<Object>`의 하위 타입은 아니다

```Java
public static void main(String[] args) {  
   List<String> strings = new ArrayList<>();  
   unsafeAdd(strings, Integer.valueOf(42));  
   String s = strings.get(0); // throw ClassCastException
   System.out.println("s = " + s);  
}  
  
private static void unsafeAdd(List list, Integer value) {  
   list.add(value);  
}
```

- 위 코드를 실행하면 Integer을 String으로 타입 변환하는 순간에 ClassCastException이 발생한다

```Java
public static void main(String[] args) {  
   List<String> strings = new ArrayList<>();  
   unsafeAdd(strings, Integer.valueOf(42));  
   String s = strings.get(0); // throw ClassCastException
   System.out.println("s = " + s);  
}  
  
private static void unsafeAdd(List<Object> list, Integer value) {  
   list.add(value);  
}
```

- 위 코드는 컴파일 조차 되지 않는다

- 실제 매개변수 타입을 신경쓰고 싶지 않다면 **비한정적 와일드카드 타입**을 사용하자

```Java
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
```

- 비한정적 와일드카드는 타입 안정적인 반면 로 타입은 그렇지 않다
	- 로 타입에는 아무 타입의 원소를 넣을 수 있다
	- `Collection<?>`은 null 외 아무 원소도 넣을 수 없다

```Java
private static void unsafeAdd(List<?> list, Integer value) {  
   list.add(value); // 예외 발생
}
```

- 비한정적 와일드 카드를 사용하면 컬렉션의 타입 불변성을 훼손하지 못하게 막을 수 있다

### 로 타입을 사용하는 예외
- class 리터럴에는 로 타입을 써야 한다
	- `List.class` `String[].class`, `int.class`는 허용하고, `List<String>.class`, `List<?>.class`는 허용하지 않는다
- instanceof 연선자에는 로 타입을 쓴다
	- 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없다
	- 로 타입이든 비한정적 와일드카드 타입이든 instanceof는 완전히 똑같이 동작한다
`
```Java
if (o instanceof Set) { // 로 타입
	Set<?> s = (Set<?>)o; // 와일드 카드 타입
}
```

