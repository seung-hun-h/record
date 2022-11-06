## ordinal 인덱싱 대신 EnumMap을 사용하라
---
- 배열이나 리스트에서 원소를 꺼낼 때 `ordinal`메서드를 사용한 코드가 있다

```Java
public class Plant {  
   enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }  
  
   final String name;  
   final LifeCycle lifeCycle;  
  
   public Plant(String name, LifeCycle lifeCycle) {  
      this.name = name;  
      this.lifeCycle = lifeCycle;  
   }  
  
   @Override  
   public String toString() {  
      return name;  
   }    
}
```
- 이제 정원에 심은 식물들을 배열 하나로 관리하고, 이들을 생애주기별로 묶어본다

```Java
public static void main(String[] args) {  
      Plant[] garden = {  
         new Plant("Basil",    LifeCycle.ANNUAL),  
         new Plant("Carroway", LifeCycle.BIENNIAL),  
         new Plant("Dill",     LifeCycle.ANNUAL),  
         new Plant("Lavendar", LifeCycle.PERENNIAL),  
         new Plant("Parsley",  LifeCycle.BIENNIAL),  
         new Plant("Rosemary", LifeCycle.PERENNIAL)  
      };  
  
      // Using ordinal() to index into an array - DON'T DO THIS!  (Page 171)  
      Set<Plant>[] plantsByLifeCycleArr = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];  
      for (int i = 0; i < plantsByLifeCycleArr.length; i++)  
         plantsByLifeCycleArr[i] = new HashSet<>();  
      for (Plant p : garden)  
         plantsByLifeCycleArr[p.lifeCycle.ordinal()].add(p);  
      // Print the results  
      for (int i = 0; i < plantsByLifeCycleArr.length; i++) {  
         System.out.printf("%s: %s%n",  
            Plant.LifeCycle.values()[i], plantsByLifeCycleArr[i]);  
      }  
}
```

- 위 코드에는 많은 문제가 있다
	- 배열과 제네릭은 호환되지 않으므롤 비검사 경고가 발생한다
	- 배열은 각 인덱스의 의미를 몰라 출력 결과에 직접 레이블을 달아야 한다
	- 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다
		- 정수는 열거 타입과 달리 타입 안전하지 않다
		- 잘못된 값을 사용하면 잘못된 동작이 묵묵시 수행되거나 운이 좋으면 ArrayIndexOutOfBoundsException이 발생한다

- 이러한 문제를 해결하기 위해서 `EnumMap`을 사용할 수 있다
```Java
EnumMap<LifeCycle, Set<Plant>> plantByLifeCycle  = new EnumMap<>(LifeCycle.class);  
for (LifeCycle lc : LifeCycle.values()) {  
   plantByLifeCycle.put(lc, new HashSet<>());  
}  
  
for (Plant plant : garden) {  
   plantByLifeCycle.get(plant.lifeCycle).add(plant);  
}  
  
System.out.println(plantByLifeCycle);
```
- 더 짧고 명로하고 안전하고 성능도 원래 버전과 비등하다
- 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천 봉쇄된다
- `EnumMap`의 성능이 `ordinal`을 사용한 코드와 비교되는 이유는 내부에서 그 배열을 사용하기 때문이다
- 내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 성능을 모두 얻어낸 것이다


### 스트림을 사용해 EnumMap 생성하기
```Java
System.out.println(Arrays.stream(garden)  
   .collect(groupingBy(p -> p.lifeCycle)));
```
- 이 코드는 `EnumMap`이 아닌 고유 구현채를 사용했기 때문에 `EnumMap`을 통해 얻을 수 있는 공간과 성능 이점이 사라진다
- 매개변수 3개짜리 `Collectors.groupingBy`메서드는 `mapFactory` 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있다

```Java
System.out.println(Arrays.stream(garden)  
   .collect(groupingBy(p -> p.lifeCycle,  
      () -> new EnumMap<>(LifeCycle.class), toSet()  
   )));
```
- 스트림을 사용하면 `EnumMap`만 사용했을 때와 다르게 동작한다
	- EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩맵을 만든다
	- 스트림을 사용하면 해당 생애주기에 속하는 식물이 있을 떄만 만든다

### 두 열거 타입 매핑 - ordinal 사용
```Java
public enum Phase {  
   SOLID, LIQUID, GAS;  
  
   public enum Transition {  
      MELT,  
      FREEZE,  
      BOIL,  
      CONDENSE,  
      SUBLIME,  
      DEPOSIT;

	  private static final Transition[][] TRANSITIONS = {
		  {null, MELT, SUBLIME},
		  {FREEZE, null, BOIL},
		  {DEPOSIT, CONDENSE, null},
	  };

	  public static Transition from(Phase from, Phase to) {
		  return TRANSITIONS[from.ordinal()][to.ordinal()];
	  }
}
```

- 컴파일러는 `ordinal`과 배열 인덱스의 관계를 알 도리가 없다
- `Phase`나 `Phase.Transition` 열거 타입을 수정하면서 상전이 `TRANSITIONS` 표를 수정하지 않거나 실수로 잘못 수정하면 런타임 오류가 발생할 수 있다
- 상전이 표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며 null로 채워지는 칸도 늘어날 것이다

### 두 열거 타입 매핑 - EnumMap 사용
```Java
public enum Phase {  
   SOLID, LIQUID, GAS;  
  
   public enum Transition {  
      MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),  
      BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),  
      SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);  
  
      private final Phase from;  
      private final Phase to;  
  
      Transition(Phase from, Phase to) {  
         this.from = from;  
         this.to = to;  
      }  
  
      private static final Map<Phase, Map<Phase, Transition>> m =  
         Stream.of(values())  
            .collect(groupingBy(t -> t.from,  
                           () -> new EnumMap<>(Phase.class),  
                           toMap(t -> t.to,  
                                t -> t,  
                                (x, y) -> y,  
                                () -> new EnumMap<>(Phase.class))  
                           )  
            );  
   }  
}
```

- `Map<Phase, Map<Phase, Transition>>`은 "이전 상태에서 '이후 상태에서 전이로의 맵'에 대응 시키는 맵"이라는 뜻이다
- 첫 번째 수집기인 `groupingBy`에서는 이전 상태를 기준으로 묶는다
- 두 번째 수집기인 `toMap`은 이후 상태를 전이에 대응시키는 `EnumMap`을 생성한다
- 두 번째 수집기에서 병합 함수는 `(x, y) -> y`는 선언만하고 실제로 쓰이지는 않는다
	- 하나의 키가 여러 값을 가지지 않음