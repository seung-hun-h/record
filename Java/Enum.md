# Enum 클래스
---
Enum은 열거형으로 불리며 **서로 연관된 집합**을 의미한다. 자바의 Enum은 아래와 같은 장점을 가진다.<br/>
1. 열거체를 비교할 때 실제 값 뿐 아니라 타입까지도 체크한다.
2. 열거체의 상수값이 재정의 되더라도 다시 컴파일할 필요가 없다
3. 인스턴스 생성과 상속을 방지하여 상수값의 타입 안정성이 보장된다
4. `enum` 키워드로 구현의도가 열거임을 확실히 알 수 있다.

## Enum의 사용
---
`enum` 키워드를 사용하여 열거체를 정의하고, `Enum이름.상수이름` 으로 사용할 수 있다.
```java
enum Color {
    RED, BLUE, GREEN
}

public class Main {
    public static void main(String[] args) {
        System.out.println(Color.RED);
        System.out.println(Color.BLUE);
        System.out.println(Color.GREEN);
    }
}
```
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/130158552-407f5eff-1868-453b-9ede-8d5747e1c4d1.png>
</p>

`enum`에 정의된 열거체들은 첫 번째부터 0, 1, ... , N으로 상수 값이 정의 된다. 불규칙적으로 상수값을 정의하고 싶으면 `()`를 추가하고 원하는 상수 값을 추가한다. 단, 상수값을 저장할 수 있는 인스턴스 변수와 생성자 `getter`를 추가해야한다.

```java
enum Color {
    RED(99), BLUE(50), GREEN(1);

    private int value;
    Color(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println(Color.RED);
        System.out.println(Color.BLUE);
        System.out.println(Color.GREEN);
    }
}

```
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/130158969-1cbbd8d1-3226-470c-994a-8dd5ea0f925c.png>
</p>

## Enum의 메소드
---
### valueOf()
>Returns the enum constant of the specified enum type with the specified name. The name must match exactly an identifier used to declare an enum constant in this type.

`enum`에 정의된 열거체와 정확이 동일한 문자열을 전달받으면 해당 열거체를 반환해주는 메소드이다
```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Color.RED);
        System.out.println(Color.BLUE);
        System.out.println(Color.GREEN);
    }
}
```
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/130159919-aa46c508-7a35-48f9-96cb-b18cbc669273.png>
</p>

### name()
>Returns the name of this enum constant, exactly as declared in its enum declaration. Most programmers should use the toString() method in preference to this one, as the toString method may return a more user-friendly name.This method is designed primarily for use in specialized situations where correctness depends on getting the exact name, which will not vary from release to release. 

`enum`에 정의되어 있는 열거체의 이름을 그대로 반환하는 메소드이다. 공식 문서에서 볼 수 있듯이 대부분의 경우 `toString()`을 사용하여 user-friendly한 이름을 반환하도록 하고, 특수한 경우에만 `name()`을 사용한다.

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Color.valueOf(\"RED\").name() = " + Color.valueOf("RED").name());
        System.out.println("Color.valueOf(\"BLUE\").name() = " + Color.valueOf("BLUE").name());
        System.out.println("Color.valueOf(\"GREEN\").name() = " + Color.valueOf("GREEN").name());
    }
}
```
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/130160450-8a6446c8-a822-4d59-afd0-606c2ae1f12e.png>
</p>

### ordinal()
>Returns the ordinal of this enumeration constant (its position in its enum declaration, where the initial constant is assigned an ordinal of zero). Most programmers will have no use for this method.

`enum`에 정의된 열거체의 정의 순서를 반환한다. 공식 문서에서는 사용하지 않는 것을 권장한다.
```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Color.RED.ordinal() = " + Color.RED.ordinal());
        System.out.println("Color.BLUE.ordinal() = " + Color.BLUE.ordinal());
        System.out.println("Color.GREEN.ordinal() = " + Color.GREEN.ordinal());
    }
}
```
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/130160662-aafffc7a-fb3d-4c56-a6b7-4ff12cc65810.png>
</p>

### values()
`enum`에 정의된 모든 열거체를 저장한 배열을 생성하여 반환하는 메소드이다. `java.lang.Enum`에 정의되어 있지는 않고 자바 컴파일러가 자동으로 추가해주는 메소드이다.
```java
public class Main {
    public static void main(String[] args) {
        for (Color color : Color.values()) {
            System.out.println(color);
        }
    }
}
```
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/130158552-407f5eff-1868-453b-9ede-8d5747e1c4d1.png>
</p>

---
**참고**
- http://tcpschool.com/java/java_api_enum
- https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Enum.html
- https://limkydev.tistory.com/50