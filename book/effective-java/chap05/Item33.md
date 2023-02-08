## 타입 안전 이종 컨테이너를 고려하라
- 제네릭은 `Set<E>`, `Map<K, V>`, `ThreadLocal<T>`, `AtomicReference<T>` 등 단일원소 컨테이너에 많이 쓰인다
- 더 유연한 수단이 필요할 때도 종종있다
	- 데이터베이스에서 가져온 열을 타입 안전하게 사용하면 좋을 것이다
	- 컨테이너 대신 키를 매개변수화하고, 컨테이너에 값을 넣거나 뺼 때 매개변수화한 키를 함께 제공하면된다
	- 이러한 방식을 타입 안전 이종 컨테이너 패턴이라 한다
- 타입 안전 이종 컨테이너란 컨테이너 대신 키를 매개변수화 하여 값의 타입이 키와 같음을 보장하는 것을 말한다

### 클래스 리터럴과 타입 토큰
- 클래스 리터럴은 `String.class`, `Integer.class` 등을 말하며, `String.class`의 타입은 `Class<String>`, `Integer.class`의 타입은 `Class<Integer>`이다
- 타입 토큰은 쉽게 말해 타입을 나타내는 토큰이며, 클래스 리터럴이 타입 토큰으로서 사용된다
	- 타입 토큰은 컴파일타입 타입 정보와 런타입 타입 정로를 알아내기 위해 메서드들이 주고 받는다

### 타입 안전 이종 컨테이너 예제
- 타입별롤 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 `Favorites` 클래스

```Java
public class Favorites {
	public <T> void putFavorite(Class<T> type, T instance);
	public <T> T getFavorite(Class<T> type);
}
```

```Java
public static void main(String[] args) {
	Favorites f = new Favorites();  
	  
	f.putFavorites(String.class, "Java");  
	f.putFavorites(Integer.class, 0xcafebabe);  
	f.putFavorites(Class.class, Favorites.class);  
	  
	String favoriteString = f.getFavorite(String.class);  
	Integer favoriteInteger = f.getFavorite(Integer.class);  
	Class<?> favoriteClass = f.getFavorite(Class.class);  
	  
	System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName());
}
```

- `Favorites`의 구현은 아래와 같이 간단하다

```Java
public class Favorites {  
   private Map<Class<?>, Object> favorites = new HashMap<>();  
   
   public <T> void putFavorites(Class<T> type, T instance) {  
      favorites.put(Objects.requireNonNull(type), type.cast(instance));  
   }  
  
   public <T> T getFavorite(Class<T> type) {  
      return type.cast(favorites.get(type));  
   }  
}
```

- `favorites`의 키에 비한정적 와일드카드를 사용해 다양한 타입을 지원할 수 있다
- `cast`는 형변환 연산자의 동적 변환이다
	- 주어진 인수가 Class 객체가 알려주는 타입의 인스턴스인지 검사하고, 맞다면 그대로 인수를 반환하고 아니면 `ClassCastException`을 던진다
	- 이를 통해 `favorites`의 불변식을 보장한다
- `Favorites` 클래스는 몇가지 문제점이 존재한다
	- 실체화 불가 타입을 사용할 수 없다
		- `List<String>` 같은 제네릭은 사용할 수 없다. `List<String>.class`는 문법오류이다.
		- 이러한 문제를 해결하기 위해 슈퍼 타입 토큰을 사용할 수 있지만, 만족스러운 우회로는 아니다
	- 타입 토큰이 비한정적이다
		- 어떤 Class는 객체로 받고있다
		- 이러한 문제를 해결하기 위해서는 한정적 타입 매개변수를 사용하면 된다

#### 애너테이션 API
- 애너테이션 API는 한정적 타입 토큰을 적극적으로 사용하고 있다
- `AnnotatedElement` 인터페이스에 선언된 메서드인 `getAnnotation()`은 대상 요소에 달려있는 애너테이션을 런타임에 읽어 오는 기능을 한다

```Java
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
```
- 이 메서드는 토큰으로 명시한 타입의 애너테이션이 대상 요소에 달려 있다면 그 애너테이션을 반환하고, 없다면 null을 반환한다
	- 즉 애너테이션된 요소는 그 키가 애너테이션 타입인 타입 안정 이종 컨테이너인 것이다
- `Class<?>` 타입의 객체를 한정적 타입 토큰을 받는 메서드에 넘기는 방법
	- 객체를 `Class<? extends Annotation>`으로 형변환할 수도 있지만, 이러한 형변환은 비검사 경고가 발생한다
	- Class는 형변환을 안전하게 수행할 수 있도록 `asSubClass`라는 메서드를 제공한다. 이는 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환한다

```Java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {

	Class<?> annotationType = null;
	try {
		annotationType = Class.forName(annotationTypeName);
	} catch(Exception ex) {
		throw new IllegalArgumentException(ex);
	} 

	return element.getAnnotation(annotationType.asSubClass(Annotation.class));
}
```

---
- [클래스 리터럴, 타입 토큰, 수퍼 타입 토큰](https://homoefficio.github.io/2016/11/30/%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%A6%AC%ED%84%B0%EB%9F%B4-%ED%83%80%EC%9E%85-%ED%86%A0%ED%81%B0-%EC%88%98%ED%8D%BC-%ED%83%80%EC%9E%85-%ED%86%A0%ED%81%B0/)
