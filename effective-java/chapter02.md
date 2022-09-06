# 2. 객체 생성과 파괴

---

## ITEM1 생성자 대신 정적 팩터리 메서드를 고려하라

> 
> 정적 팩터리 메서드와 public 생성자느 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다. 그렇다고 하더라도 정적 팩터리를 사용하는게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고치자
> 


- 클래스는 생성자와 별도로 정적 팩터리 메서드르 제공할 수 있다
- 정적 팩터리 메서드는 해당 클래스의 인스턴스를 반환하는 정적 메서드이다

```Java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

### 정적 팩터리 메서드 장점

**1. 이름을 가질 수 있다**
   - 정적 팩터리 메서드는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다
   - 하나의 시그니처로는 생성자를 하나만 만들 수 있다
   - 매개변수의 순서를 변경하는 등 한 생성자를 새로 추가할 수 있지만, 각 생성자가 어떤 역할을 하는지 구분하기 어렵다
   - 시그니처가 같은 생성자가 여러개 필요할 것 같으면, 생성자를 정적 팩터리 메서드로 바꾸고 이름을 잘 지어주어야 한다
     
**2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다**
   - 인스턴스를 미리 만들어 놓거나 인스턴스를 캐싱할 수 있다
   - 크기가 큰 객체가 자주 요청되는 상황에서 성능을 높일 수 있다
   - 인스턴스 통제 클래스를 구현할 수 있다. 인스턴스 통제 클래스는 언제 어느 인스턴스를 살아 있게할 지를 철저하게 통제하는 클래스를 말한다
   - 인스턴스 통제를 통해 싱글턴, 인스턴스화 불가 클래스를 만들 수 있다
   - 그리고 불변 값 클래스에서 동치인 인스턴스가 단 하나 뿐임을 보장할 수 있다(a == b인 경우에만 a.equals(b))
     
**3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다**
   - 반환할 객체의 클래스를 자유롭게 선택해 유연성을 제공한다
   - 자바 컬렉션 프레임워크는 핵심 인터페이스들에 수정 불가나 동기화 등의 기능을 덧붙인 45개의 유틸리티 구현체를 제공한다
   - 컬렉션은 45개의 유틸리티 구현체를 제공하지 않기 때문에, 프로그래머는 명시한 인터페이스대로 쉽게 사용할 수 있다
   - 컬렉션의 Collections는 인터페이스를 반환하는 정적 메서드가 구현된 동반 클래스이다. 자바 8 부터는 인터페이스에 정적 메서드를 허용하기 때문에 동반 클래스를 구현할 필요가 없다
   - 하지만 정적 필드와 정적 멤버 클래스는 public 이어야 하기 때문에 세부 구현을 노출시키지 않으려면 동반 클래스를 구현해야 한다
     
**4. 입력 매개 변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다**
   - 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다
   - EnumSet 클래스는 원소가 64개 이하이면 RegularEnumSet, 65개 이상이면 JumboEnumSet을 반환한다
   - 클라이언트는 정적 팩토리 메서드의 구체적인 타입을 알 필요가 없고, EnumSet의 하위 타입이기만 하면 된다

```Java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {  
    Enum<?>[] universe = getUniverse(elementType);  
    if (universe == null)  
        throw new ClassCastException(elementType + " not an enum");  
  
    if (universe.length <= 64)  
        return new RegularEnumSet<>(elementType, universe);  
    else        
	    return new JumboEnumSet<>(elementType, universe);  
}
```

**5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다**(?)
- 이런 유연함은 서비스 제공자 프레임 워크의 근간이 된다. 대표적으로 JDBC가 있다
- 서비스 제공자 프레임 워크의 핵심 컴포넌트 3가지 + 종종 쓰이는 1가지
	- 서비스 인터페이스: 구현체의 동작을 정의
		- `Connection`
	- 제공자 등록 API: 제공자가 구현체를 등록할 떄 사용
		- `DriverManager.registerDriver`
	- 서비스 접근 API: 클라이언트가 서비스의 인스턴스를 얻을 때 사용
		- `DriverManager.getConnection`
	- 서비스 제공자 인터페이스: 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명해준다
		- `Driver`

```Java
private static Connection getConnection(  
    String url, java.util.Properties info, Class<?> caller) throws SQLException {  
    
    ClassLoader callerCL = caller != null ? caller.getClassLoader() : null;  
    
    if (callerCL == null || callerCL == ClassLoader.getPlatformClassLoader()) {  
        callerCL = Thread.currentThread().getContextClassLoader();  
    }  
  
    if (url == null) {  
        throw new SQLException("The url cannot be null", "08001");  
    }  
  
    for (DriverInfo aDriver : registeredDrivers) {  
		if (isDriverAllowed(aDriver.driver, callerCL)) {  
            try {  
                Connection con = aDriver.driver.connect(url, info);  
                if (con != null) {  
                    return (con);  
                }  
            } catch (SQLException ex) {  
                if (reason == null) {  
                    reason = ex;  
                }  
            }  
  
        } 
    }  
  
    if (reason != null)    {  
        throw reason;  
    }  
  
    throw new SQLException("No suitable driver found for "+ url, "08001");  
}
```
- `getConnection`을 작성하는 시점에는 `Connection`의 구현체는 모른다
- DB 드라이버에 따라 `Connection`의 구현체가 있을 것이고, `getConnection`은 전달받은 인자를 통해서 적절한 `Connection`  구현체를 반환한다
- 그러니까 Connection의 구현체를 나중에 만들 수 있다

### 정적 팩터리 메서드 단점
1. 상속을 하려면 public, protected 생성자가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다
   - 이 제약은 상속보다는 컴포지션을 사용하도록 유도하고, 불변 타입으로 만드려면 이 제약을 지켜야한다는 점에서 장점일 수 있다
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다
   - API 설명에 명확히 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화 할 방법을 찾아야 한다

### 정적 팩터리 메서드의 일반적인 이름
- from: 매개변수 하나를 받아서 해당 타입의 인스턴스를 반환
- of: 여러 매개변수를 받아서 인스턴스를 반환
- valueOf: from, of의 더 자세한 버전
- instance, getInstance: 매개변수를 받으면 매개변수로 명시한 인스턴스를 반환. 같은 인스턴스를 보장하지는 않음
- create, newInstance: 매번 새로운 인스턴스 반환
- getType: getInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다
- newType: newInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 맥터리 메서드를 정의할 때 쓴다
- type: getType과 newType의 간결한 버전

---

## ITEM2 생성자에 매개변수가 많다면 빌더를 고려하라

>
>생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 낫다
>매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고 자바빈즈보다 안전하다
>

- 정적 팩터리와 생성자는 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 단점이 있다
- 선택적 매개변수가 많을 때 대응 방안은 점층적 생성자 패턴, 자바빈즈 패턴, 빌더 패턴을 사용할 수 있다

### 점층적 생성자 패턴(Telescoping constructor pattern)
- 점층적 생성자 패턴은 선택 매개변수의 개수를 늘려가며 생성자를 만드는 패턴이다

```Java
public class NutritionFacts {  
   private final int servingSize;  
   private final int servings;  
   private final int calories;  
   private final int fat;  
   private final int sodium;  
   private final int carbohydrate;  
  
   public NutritionFacts(int servingSize, int servings) {  
      this(servingSize, servings, 0);  
   }  
  
   public NutritionFacts(int servingSize, int servings, int calories) {  
      this(servingSize, servings, calories, 0);  
   }  
  
   public NutritionFacts(int servingSize, int servings, int calories, int fat) {  
      this(servingSize, servings, calories, fat, 0);  
   }  
  
   public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {  
      this(servingSize, servings, calories, fat, sodium, 0);  
   }  
  
   public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {  
      this.servingSize = servingSize;  
      this.servings = servings;  
      this.calories = calories;  
      this.fat = fat;  
      this.sodium = sodium;  
      this.carbohydrate = carbohydrate;  
   }  
}
```

- 해당 클래스의 인스턴스를 생성하기 위해서는 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 사용하면 된다
- 매개 변수가 늘어나면 클라이언트 코드를 작성하거나 읽기 어려워진다
	- 사용자가 설정하기 원치 않는 매개변수에도 값을 지정해주어야 한다

### 자바빈즈 패턴(JavaBeans pattern)
- 매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식이다

```Java
package item2.setter;  
  
public class NutritionFacts {  
   private int servingSize;  
   private int servings;  
   private int calories;  
   private int fat;  
   private int sodium;  
   private int carbohydrate;  
  
   public void setServingSize(int servingSize) {  
      this.servingSize = servingSize;  
   }  
  
   public void setServings(int servings) {  
      this.servings = servings;  
   }  
  
   public void setCalories(int calories) {  
      this.calories = calories;  
   }  
  
   public void setFat(int fat) {  
      this.fat = fat;  
   }  
  
   public void setSodium(int sodium) {  
      this.sodium = sodium;  
   }  
  
   public void setCarbohydrate(int carbohydrate) {  
      this.carbohydrate = carbohydrate;  
   }  
}
```

- 점층적 생성자 패턴을 사용할 때보다 인스턴스를 만들기 쉽고 그 결과 코드가 더 읽기 쉬워졌다
- 하지만 자바빈즈 패턴에서는 객체 하나를 만들기 위해 **여러 메서드를 호출**해야 하고, 객체가 완전히 생성되기 전까지는 **일관성이 무너진 상태**에 놓이게 된다
- 자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없다

### 빌더 패턴
- 점층적 생성자 패턴의 안정성과 자바 빈즈 패턴의 가독성을 겸비한 패턴이다
- 클라이언트가 필요한 객체를 직접 만드는 대신 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다
- 그 다음 빌더 객체가 제공하는 일정의 세터 메서드들로 원하는 선택 매개변수들을 설정한다
- 마지막으로 매개변수가 없는 `build()` 메서드를 호출해 필요한 객체를 얻는다

```Java
public class NutritionFacts {  
   private final int servingSize;  
   private final int servings;  
   private final int calories;  
   private final int fat;  
   private final int sodium;  
   private final int carbohydrate;  
  
   public NutritionFacts(Builder builder) {  
      this.servingSize = builder.servingSize;  
      this.servings = builder.servings;  
      this.calories = builder.calories;  
      this.fat = builder.fat;  
      this.sodium = builder.sodium;  
      this.carbohydrate = builder.carbohydrate;  
   }  
  
   public static class Builder {  
      // 필수 매개 변수  
      private final int servingSize;  
      private final int servings;  
  
      // 선택적 매개 변수  
      private int calories;  
      private int fat;  
      private int sodium;  
      private int carbohydrate;  
  
      public Builder(int servingSize, int servings) {  
         this.servingSize = servingSize;  
         this.servings = servings;  
      }  
  
      public Builder calories(int calories) {  
         this.calories = calories;  
         return this;      }  
  
      public Builder fat(int fat) {  
         this.fat = fat;  
         return this;      }  
  
      public Builder sodium(int sodium) {  
         this.sodium = sodium;  
         return this;      }  
  
      public Builder carbohydrate(int carbohydrate) {  
         this.carbohydrate = carbohydrate;  
         return this;      }  
  
      public NutritionFacts build() {  
         return new NutritionFacts(this);  
      }  
   }  
}
```

- 잘못된 매개변수를 최대한 일찍 발견하기 위해 빌더의 생성자와 메서드에 입력 매개변수를 검사한다
- 그리고 build 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식을 검사한다
- 불변식을 보장하려면 빌더로부터 매개변수를 복사한 후 해당 객체 필드들도 검사해야 한다
- 잘못된 점을 찾으면 **IllegalArgumentException**을 던진다

**빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다**
- 추상 클래스는 추상 빌더를, 구체 클래스는 구체 빌더를 갖게 한다

```Java
public abstract class Pizza {  
   public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }  
   final Set<Topping> toppings;  
  
   abstract static class Builder<T extends Builder<T>> {  
      EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);  
  
      public T addTopping(Topping topping) {  
         toppings.add(Objects.requireNonNull(topping));  
         return self();  
      }  
  
      abstract Pizza build();  
  
      protected abstract T self();  
   }  
  
   Pizza(Builder<?> builder) {  
      this.toppings = builder.toppings.clone();  
   }  
}

public class NyPizza extends Pizza {  
   public enum Size { SMALL, MEDIUM, LARGE }  
   private final Size size;  
  
   public static class Builder extends Pizza.Builder<Builder> {  
      private final Size size;  
  
      public Builder(Size size) {  
         this.size = Objects.requireNonNull(size);  
      }  
  
      @Override  
      public NyPizza build() {  
         return new NyPizza(this);  
      }  
  
      @Override  
      protected Builder self() {  
         return this;  
      }  
   }  
  
   NyPizza(Builder builder) {  
      super(builder);  
      size = builder.size;  
   }  
}

public class Calzone extends Pizza {  
   private final boolean sauceInside;  
  
   public static class Builder extends Pizza.Builder<Builder> {  
      private boolean sauceInside = false;  
  
      public Builder sauceInside() {  
         this.sauceInside = true;  
         return this;      }  
  
      @Override  
      public Calzone build() {  
         return new Calzone(this);  
      }  
  
      @Override  
      protected Builder self() {  
         return this;  
      }  
   }  
  
   Calzone(Builder builder) {  
      super(builder);  
      this.sauceInside = builder.sauceInside;  
   }  
}
```

- `Pizza.Builder`는 재귀적 타입 한정을 사용하는 제네릭 타입이다
- 추상 메서드인 `self`를 더해 하위 클래스에서는 형변환하지 않고도 메서드 연쇄를 지원할 수 있다(?)
- 하위 클래스의 `build` 메서드는 구체 하위 타입을 반환하도록 한다
	- 상위 타입이 아닌 하위 타입을 반환하는 기능을 공변 반환 타이핑이라 한다
	- 클라이언트는 형변환에 신경쓰지 않고 빌더를 사용할 수 있다
- 빌더의 생성 비용은  크지 않지만 성능이 민감한 상황인 경우에는 문제가 될 수 있다
- 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다
- 초반에는 생성자나 정적 팩터리 메서드로 시작하고 나중에 빌더로 바꿀 수 있지만 문제가 발생할 수 있으므로 애초에 빌더로 시작하는 것이 나을 떄가 있다