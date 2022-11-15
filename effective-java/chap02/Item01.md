## ITEM01 생성자 대신 정적 팩터리 메서드를 고려하라

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

- DBMS
	- MySQL, Oracle, MariaDB .. 여러가지(벤더)
	- 벤더사마다 Connection 객체, DriverManager 다르다

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