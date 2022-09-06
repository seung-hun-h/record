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