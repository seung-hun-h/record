## Java 8에서 추가된 기능들
---
### default Method & static Method
- Java 8 이전까지 인터페이스에는 추상 메서드만 정의 가능 했다.
- 인터페이스에 기능을 추가하기 위해서는 추상 메서드를 추가 정의하면, 모든 구현 클래스에서 메서드를 구현해야하기 때문에 번거로워진다.
- 따라서 필수적으로 구현하지 않아도 되는 default 메서드를 추가할 수 있게 되었다.
- 그리고 static 메서드도 추가할 수 있게 되었다(ex: Collections.sort()).

### Functional Interface
- static, default 메서드를 제외하고, 단 하나의 추상 메서드만 가지는 인터페이스를 함수형 인터페이스라고 한다.
- @FunctionalInterface 어노테이션을 달 수 있다

### Lambda 표현식와 메서드 참조
- Java 8 이전에는 클래스와 메서드가 인자로 전달 될 수 없었다.
- 따라서 클래스와 메서드는 이급 시민이었다.
- 클래스와 메서드를 일급 시민(인자로 전달 가능)으로 취급할 수 있게 한 것이 람다와 메서드 참조이다
- 람다 표현식은 함수형 인터페이스에만 사용 가능하며, 익명 클래스에서 불필요한 부분을 제거하여 간결하게 표현할 수 있게 한다
- 메서드 참조는 람다 표현식에 전달되는 값에 변화가 없을 경우 값을 생략할 수 있는 것이다.
    - 메서드 참조를 사용하면 개발자의 개입을 차단하여 안정성을 얻을 수 있다.
**람다 표현식 예**  
```java
public static void main(String[] args) {
    List<Integer> list = new ArrayList<>(Arrays.asList(5, 1, 6, 2, 3, 8));

    // 익명 클래스 사용
    Collections.sort(list, new Comparator<>() {
        @Override
        public int compare(Integer a1, Integer a2) {
            return a1 - a2;
        }
    });

    //람다 표현식 사용
    Collections.sort(list, (a1, a2) -> a1 - a2);
}
```

**메서드 참조 예**
```java
public class Main {
    public static void main(String[] args) {
        Printer printer = new Main().new Printer;

        // 익명 클래스
        printer.print("1", new Mapper() {
            int map(String str) {
                return Integer.parseInt(str);
            }
        });

        // 람다 표현식
        printer.print("1", (str) -> Integer.parseInt(str));
    
        // 메서드 참조
        // 전달 받은 String 타입의 인자를 
        // 수정하지 않으므로 메서드 참조가 가능하다
        printer.print("1", Integer::parseInt);
    }
    interface Mapper {
        int map(String str);
    }

    public class Printer {
        void print(String str, Mapper mapper) {
            System.out.println(mapper.map(str));
        }
    }
}