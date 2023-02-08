## 다른 타입이 적절하다면 문자열 사용을 피하라
---

### 문자열은 다른 값 타입을 대신하기 적합하지 않다
- 파일, 네트워크, 키보드 입력으로 데이터를 받을 때 주로 문자열을 사용한다
- 하지만 입력받을 데이터가 진짜 문자열일때만 그렇게 하는 것이 좋다
- 기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고, 없다면 새로 작성하자

### 문자열은 열거 타입을 대신하기에 적합하지 않다
- 상수를 열거할 때는 문자열보다는 열거 타입이 월등히 낫다

### 문자열은 혼합 타입을 대신하기에 적합하지 않다
- 여러 요소가 혼합된 데이터를 하나의 문자열로 표현하는 것은 대체로 좋지 않은 생각이다
- 각 요소를 개별로 접근하려면 문자열을 파싱해야 해서 느리고, 귀찮고, 오류 가능성도 커진다
- 차라리 전용 클래스를 새로 만드는 편이 낫다
	- 이런 클래스는 보통 private 정적 멤버 클래스로 선언한다

### 문자열은 권한을 표현하기에 적합하지 않다
-  권한을 문자열로 표현하는 경우가 종종있다
- 스레드별로 지역변수 기능을 설계한다고 가정해보자

```Java
public class ThreardLocal {
	private ThreadLocal() { } // 객체 생성 불가

	public static void set(String key, Object value);
	public static Object get(String key);
}
```

- 이 방식의 문제는 스레드 구분용 문자열 키가 전역 이름 공간에서 공유된다는 것이다
	- 의도한 대로 동작하기 위해서는 각 클라이언트가 고유한 키를 제공해야 한다
- 두 클라이언트가 소통하지 못해 같은 키를 쓰기로 한다면 의도치 않게 같은 변수를 공유하게 된다
- 악의적인 클라이언트라면 의도적으로 같은 키를 사용하여 다른 클라이언트의 값을 가져올 수도 있다
- 문자열 대신 위조할 수 없는 키를 사용하면 해결된다

```Java
public class ThreardLocal {
	private ThreadLocal() { } // 객체 생성 불가

	public static class Key {
		Key() { }
	}

	public static void set(Key key, Obejct value);
	public static Object get(Key key);
}
```

- 이 방식은 문자열 기반 API의 문제점을 해결 해주지만, 개선할 여지가 있다
	- set, get은 이제 정적 메서드일 이유가 없으므로, Key 클래스의 인스턴스 메서드로 변경하자
	- Key는 더이상 지역변수를 구분하기 위한 키가 아니라 그 자체가 스레드의 지역변수가 된다
- 톱 레벨 클래스인 ThreadLocal은 별 달리 하는 일이 없으므로 치워버리고, Key의 이름을 ThreadLocal로 바꾸자

```Java
public final class ThreadLocal {
	public ThreadLocal();
	public void set(Object value);
	public Object get();
}
```

- 이 방식에서는 get으로 얻은 Object를 실제 타입으로 형 변환해야 하므로 타입 안전하지 않다
- 매개변수화 타입으로 선언하면 이 문제는 간단히 해결된다

```Java
public final class ThreadLocal<T> {
	public ThreadLocal();
	public void set(T value);
	public T get();
}
```


