## 적시에 방어적 복사본을 만들라
---
- 클라이언트가 불변식을 깨뜨리려 혈안이 되어 있다는 가정하에 방어적으로 프로그래밍 해야 한다

```Java
public class Period {  
   private final Date start;  
   private final Date end;  
  
   public Period(Date start, Date end) {  
      if (start.compareTo(end) > 0) {  
         throw new IllegalArgumentException(String.format("%s가 %s보다 늦다", start, end));  
      }  
        
      this.start = start;  
      this.end = end;  
   }  
  
   public Date getStart() {  
      return start;  
   }  
  
   public Date getEnd() {  
      return end;  
   }  
}
```

- 일단 `Date`는 이제 사용하면 안된다

### 생성자
- 외부 공격으로 부터 `Period` 인스턴스 내부를 보호하기 위해서는 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사 해야 한다
  
```Java
public Period(Date start, Date end) {  	
  this.start = new Date(start.getTime());  
  this.end = new Date(end.getTime());  
  
  if (this.start.compareTo(this.end) > 0) {  
	 throw new IllegalArgumentException(String.format("%s가 %s보다 늦다", start, end));  
  }
}  
```

- 매개변수 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성 검사를 한 것에 주목하자
	- 멀티스레딩 환경에서 원본 객체의 유효성을 검사한 후 복사본을 만드는 찰나의 취약한 순간에 다른 스레드가 원본 객체를 수정할 위험이 있기 때문이다
		- 인자로 넘어온 start, end가 변경될 위험이 있다
- 방어적 복사에 `Date`의 `clone`을 사용하지 않은 점에도 주목하자
	- `Date`는 `final`이 아니므로 `clone`이 `Date`가 정의한 것이 아닐 수 있다
	- 다른 하위 클래스가 악의적으로 재정의 했을 수도 있다
	- 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 `clone`을 사용하면 안된다


### 접근자
- 접근자 메서드가 내부의 가변 정보를 직접 드러내면, `Period`의 내부 정보를 변경할 수 있다
- 이 문제는 접근자가 가변 필드의 방어적 복사본을 반환하면 된다

```Java
public Date getStart() {  
   return new Date(start.getTime());  
}  
  
public Date getEnd() {  
   return new Date(end.getTime());  
}
```
- 생성자와 달리 접근자에서는 `clone`을 사용해도 된다
	- Period가 가진 객체가 `java.util.Date`임이 확실하기 때문이다
	- 그렇지만 인스턴스를 복사하는 데는 일반적으로 생성자나 정적 팩터리를 사용하는 것이 좋다


### 방어적 복사를 하는 다른 이유
- 매개변수를 방어적으로 복사하는 목적이 불변객체를 만드는 것만이 아니다
- 변경될 수 있는 객체가 클래스에 넘겨진 뒤 임의로 변경되어도 해당 클래스에 문제가 발생하지 않는지 생각해봐야 한다
	- 클라이언트가 넘겨준 객체를 내부의 `Set`이나 `Map` 인스턴스의 키로 사용한다면, 추후 그 객체가 변경된다면 `Set`과 `Map`의 불변식이 깨질 수 있다
- 클래스가 불변이든 가변이든, 가변인 내부 객체를 클라이언트에 반환할 때는 반드시 심사숙고 해야 한다
	- 안심할 수 없으면 그냥 방어적 복사본을 반환해라

### 방어적 복사를 생략할 수 있는 경우
- 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략할 수 있다
- 가변 매개변수를 그대로 넘기는 것이 그 객체의 통제권을 명백히 이전하는 것을 뜻하기도 한다
	- 통제권을 이전하는 메서드를 호출하는 클라이언트는 해당 객체를 더 이상 직접 수정하는 일이 없다고 약속해야 한다
- 방어적 복사를 생략해도 되는 상황은 해당 클래스와 그 클라이언트가 상호 신뢰할 수 있을 때, 혹은 불변식이 깨지더라도 그 영향이 오직 호출한 클라이언트로 국한 될 때로 한정해야 한다