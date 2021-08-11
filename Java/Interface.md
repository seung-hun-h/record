## Interface
---
인터페이스는 모든 메서드가 추상 메서드인 클래스이다.

### 인터페이스의 기능<br/>
- **구현을 강제한다**<br/>
    추상 메서드의 구현을 강제하여 개발자에게 가이드라인을 제공한다.
- **다형성을 제공한다**<br/>
    다양한 구현체를 통해서 다형성을 제공한다.
- **결합도를 낮춘다**
    구현체에 의존할 경우 결합도가 강하다고 하며, 추상체에 의존할 경의 결합도가 낮아진다고 한다. 따라서 인터페이스를 사용하면 결합도를 낮출 수 있다

### DI(Dependency Injection)
- A 클래스가 B 클래스의 인스턴스를 사용하여 로직을 수행할 경우 '**A 클래스는 B에 의존한다**'고 표현한다


```java
public class Member {
    Long id;
    String email;
    ...
}

interface MemberRepository {
    void save(Member member);
    Member findById(Long id);
    Member findByEmail(String email);
    ...
}

public class MemoryMemberRepository implements MemberRepository {
    ...
}

public class DBMemberRepository implements MemberRepository {
    ...
}

```

- Member 클래스는 인스턴스 멤버로 id와 email을 가진다. 
- MeberRepository는 save, findById, findByEmail 추상 메소드를 가진다.
- MemoryMemberRepository는 메모리상에 Member 정보를 저장하고 접근하며, DBMemberRepository는 DB에 Member 정보를 저장하고 접근한다. 

```java

public class MemberService {
    private final MemberRepository memberRepository = new MemoryMemberRepository();

    // private final MemberRepository memberRepository = new MemoryMemberRepository();
    ...
```

- MemberService는 비즈니스 로직을 수행하는 클래스로 로직을 수행할 때 MemberRepository 객체를 사용한다.
- **따라서 현재 MemberService는 MemberRepository에 의존한다.**
- 하지만 MemberService는 현재 MemberRepository의 객체를 직접 생성하기 때문에 결합도가 강하다.
- 결합도를 낮추기 위해 별도의 설정 클래스를 생성해서 외부에서 의존성을 설정해준다.

```java

public class MemberService {
    private final MemberRepository memberRepository;

   MemberService (MemberRepository memberRepository) {
       this.memberRepository = memberRepository;
   }
    ...
```
```java
public class Config {

    MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    MemberService getMemberService() {
        return new MemberService(memberRepository());
    }
}
```

- 생성자를 통해서 MemberService의 의존을 주입 받게 했고, 외부 설정 클래스에서 의존성을 주입해주었다.
- 이를 **의존성 주입(DI)**이라 한다.
- 이제 MemberService 클래스는 MemberRepository의 구현체를 직접 생성하지 않아도 되어 추상체에만 의존하게 되었다.
- 따라서 결합도가 낮아졌다.