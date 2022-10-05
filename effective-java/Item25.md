## ITEM25 톱클래스는 한 파일에 하나만 담으라
- 소스 파일 하나에 톱레벨 클래스 여러 개를 선언하더라도 자바 컴파일러는 불평하지 않는다
- 하지만 이렇게 하면 한 클래스를 여러 가지로 정의할 수 있으며, 그 중 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하냐에 따라 달라진다

```Java
public class Main {  
   public static void main(String[] args) {  
      System.out.println(Utensil.NAME + Dessert.NAME);  
   }  
}
```

**Utensil.java**
```Java
class Utensil {  
   static final String NAME = "pan";  
}  
  
class Dessert {  
   static final String NAME = "cake";  
}
```

**Dessert.java**
```Java
class Utensil {  
   static final String NAME = "pan";  
}  
  
class Dessert {  
   static final String NAME = "cake";  
}
```

- 운 좋게 javac Main.java Dessert.java 명령으로 컴파일 한다면 컴파일 오류가 나고 Utensil과 Dessert 클래스를 중복 정의했다고 알려줄 것이다
- 한 편 javac Main.java나 javac Main.java Utensil.java 명령으로 컴파일 하면 Dessert.java 파일을 작성하기 전 처럼 pancake을 출력한다
- 컴파일러에 어떤 소스 파일을 먼저 건네냐에 따라 동작이 달라지므로 바로잡아야 한다
- 이를 해결하기 위해서는 한 파일에 하나의 톱클래스를 작성하면된다
- 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용하는 방법을 고민해보자
	- 읽기 좋고, 접근 범위도 최소화할 수 있다

---
실제로 해보면 컴파일 오류가 나지 않는다

#### javac Main.java Utensil.java
**Utensil.class**
```Java
class Utensil {  
    static final String NAME = "pan";  
  
    Utensil() {  
    }  
}
```


**Dessert.class**
```Java
class Dessert {  
    static final String NAME = "cake";  
  
    Dessert() {  
    }  
}
```

**Main.class**
```Java
public class Main {  
    public Main() {  
    }  
  
    public static void main(String[] var0) {  
        System.out.println("pancake");  
    }  
}
```

#### javac Main.java Dessert.java
**Utensil.class**
```Java
class Utensil {  
    static final String NAME = "pot";  
  
    Utensil() {  
    }  
}
```


**Dessert.class**
```Java
class Dessert {  
    static final String NAME = "pie";  
  
    Dessert() {  
    }  
}
```

**Main.class**
```Java
public class Main {  
    public Main() {  
    }  
  
    public static void main(String[] var0) {  
        System.out.println("potpie");  
    }  
}
```
