## ITEM14 Comparable을 구현할지 고려하라
>
> 순서를 고료해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하라. compareTo 메서드를 구현할때는 비교 연산자는 사용하지 않아야 한다. 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.
>

- Comparable의 `compareTo`는 두 가지 성격만 제외하면 equals와 같다
	1. compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있고, 제네릭하다
	2. Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻한다

- Comparable을 구현하면 이 인터페이스를 활용하는 수많은 제네릭 알고리즘과 컬렉션의 힘을 이용할 수 있다
- 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자

### compareTo 메서드의 일반 규약

> 
   이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다. 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
> 
> 다음 설명에서 sgn 표기는 수학에서 말하는 부호 함수를 뜻하며 음수, 0, 양수일때 -1, 0, 1 을 반환하도록 정의 했다
> 
> 1. Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다
> 2. Comparable을 구현한 클래스는 추이성을 보장해야 한다
> 3. Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))
> 4. 이번 권고가 필수는 아니지만 꼭 지키는 것이 좋다. (x.compareTo(y) == 0) == (x.equals(y))여야 한다
>     

- equals 메서드와 달리 compareTo는 타입이 다른 객체를 신경쓰지 않아도 된다. 타입을 다르면 ClassCastExcetion을 던지면 된다
- compareTo 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못한다
	- TreeSet, TreeMap
	- 정렬 알고리즘을 사용하는 유틸 클래스인 Collections, Arrays

- equals와 마찬가지로 compareTo도 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지킬 방법이 없다
- 이를 해결하기 위해 컴포지션을 사용하고, 내부 인스턴스를 반환하는 뷰 메서드를 제공하면 된다

- 마지막 규약은 필수는 아니지만 꼭 지키기를 권장한다
	- 정렬된 컬렉션들은 동치성을 비교할 때 equals대신 compareTo를 사용한다
```Java
public static void main(String[] args) {  
   BigDecimal bigDecimal1 = new BigDecimal("1.0");  
   BigDecimal bigDecimal2 = new BigDecimal("1.00");  
  
   Set<BigDecimal> hashSet = new HashSet<>() {  
      {  
         add(bigDecimal1);  
         add(bigDecimal2);  
      }  
   };  
  
   Set<BigDecimal> treeSet = new TreeSet<>() {  
      {  
         add(bigDecimal1);  
         add(bigDecimal2);  
      }  
   };  
  
   System.out.println("hashSet = " + hashSet); 
   System.out.println("treeSet = " + treeSet);  
}

/**
hashSet = [1.0, 1.00]
treeSet = [1.0]
*/
```

### compareTo 작성 요령
- 몇 가지 차이점을 제외하면 equals와 동일하다
- Comparable는 타입을 인수로 받는 제네릭이므로 컴파일 타임에 타입이 정해진다
	- 입력 인수의 타입을 확인하거나 형변환 할 필요가 없다
- null을 인수로 넣어 호출하면 NullPointerException을 던저야 한다
- 객체의 참조 필드를 비교하려면 compareTo를 재귀적으로 호출한다
	- Comparable을 구하지 않은 참조 필드는 Comparator를 대신 사용한다
- compareTo에서 비교 연산자(<, >)는 사용하지 않는다
- 핵심 필드가 여러개라면 중요한 필드를 먼저 비교한다

### 메서드 연쇄 방식 비교자 생성
- 자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다

```Java
private static final Comparator<PhoneNumber> COMPARATOR = 
		comparingInt((PhoneNumber pn -> pn.areaCode)
					.thenComparingInt(pn -> pn.prefix)
					.thenComparingInt(pn -> pn.lineNum))

public int compareTo(PhoneNumber pn) {
	return COMPARATOR.compare(this, pn);
}
```

- int형 뿐아니라 float이나 double도 타입에 맞는 전용 비교자 생성 메서드를 사용하면 된다
- 객체 참조용 비교자 생성메서드도 comparing, thenComparing 두 가지가 존재한다