## 다중정의는 신중히 사용하라
```java
public class CollectionClassifier {  
   public static String classify(Set<?> s) {  
      return "집합";  
   }  
  
   public static String classify(List<?> s) {  
      return "리스트";  
   }  
  
   public static String classify(Collection<?> s) {  
      return "그 외";  
   }  
  
   public static void main(String[] args) {  
      Collection<?>[] collections = {  
         new HashSet<String>(),  
         new ArrayList<Integer>(),  
         new HashMap<String, String>().values()  
      };  
  
      for (Collection<?> collection : collections) {  
         System.out.println(classify(collection));  
      }  
   }  
}
```
- "집합", "리스트", "그 외"를 출력할 것 같지만, "그 외"만 세 번 출력한다
- 이는 다중정의된 세 `classify`중 어느 메서드가 호출될 지 컴파일 타임에 결정되기 때문이다
	- 컴파일 타임에는 for만 안의 collection은 항상 `Collection<?>` 타입이다
	- 런타임에는 매번 바뀌지만, 현재 코드에 영향을 주지는 못한다

- **재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택된다**

```Java 
public class Overriding {  
   static class Wine {  
      String name() {  
         return "포토주";  
      }  
   }  
  
   static class SparklingWine extends Wine {  
      @Override  
      String name() {  
         return "발포성 포도주";  
      }  
   }  
  
   static class Champagne extends Wine {  
      @Override  
      String name() {  
         return "샴페인";  
      }  
   }  
  
   public static void main(String[] args) {  
      List<Wine> wines = 
			      List.of(new Wine(), new SparklingWine(), new Champagne());  
      for (Wine wine : wines) {  
         System.out.println(wine.name());  
      }  
   }  
}
```

- 위 프로그램은 예상한 것처럼 "포도주", "발포성 포도주", "샴페인"을 출력한다

### 다중정의는 컴파일타임 타입으로 결정된다
- 다중정의된 메서드 사이에는 객체의 런타임 타입은 전혀 중요하지 않다
- 선택은 컴파일타임에, 오직 매개변수의 컴파일타임 타입에의해 이뤄진다

### 다중정의가 혼동을 일으키는 상황은 피하자
- 헷갈릴 수 있는 코드는 작성하지 않는 것이 좋다
- 안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자
	- 가변인수를 사용하는 메서드라면 다중정의를 아예 하지 말아야 한다
- 다중정의를 하는 대신 메서드 이름을 변경하는 길은 언제나 열려있다
- 다중정의할 때 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다
	- 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생긴다
