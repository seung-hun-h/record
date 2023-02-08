## ITEM08 finalizer와 cleaner 사용을 피하라

>
>cleaner(자바 8까지는 finalizer)는 안정망 역할이나 중요하지 않는 네이티브 자원 회수용으로만 사용하자, 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다
>

- 자바는 두 가지 객체 소멸자를 제공한다
- `finalizer`는 예측할 수 없고 상황에 따라 위험할 수 있어 일반적으로 불필요하다
- `cleaner`는 finalizer보다 덜 위험하지만 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다

### finalizer와 cleaner는 즉시 수행을 보장하지 않는다
- finalizer와 cleaner는 제때 실행되어야 하는 작업은 절대 할 수 없다
- 자바 언어 명세는 finalizer와 cleaner의 수행 시점 뿐 아니라 수행 여부조차 보장하지 않는다
- 상태를 영구적으로 수정하는 작업에서는 절다 finalizer와 cleaner를 사용하면 안된다
- `System.gc`. `System.runFinalization`은 finalizer와 cleaner가 실행될 가능성은 높여주지만, 보장해주지는 않는다
- `System.runFinalizersOnExit`, `Runtime.runFinalizersOnExit`은 실행을 보장해주기 위해 나타났지만, 심각한 결함을 가지고 있다

```Text
## Problem

`Runtime::runFinalizersOnExit` is inherently unsafe. It has been deprecated since 1.2. It has also been deprecated for removal in Java SE 9.

Calling this method may result in finalizers being called on live objects while other threads are concurrently manipulating those objects, resulting in erratic behavior or deadlock. While this problem could be prevented if the class whose objects are being finalized were coded to "defend against" this call, most programmers do not defend against it. They assume that an object is dead at the time that its finalizer is called.

## Solution

Remove `Runtime::runFinalizersOnExit` and `System::runFinalizersOnExit` methods.

Update the spec of `Runtime::addShutdownHook`, `Runtime::exit`, `Runtime::halt` describing the shutdown sequence and drop the phase about invoking all finalizers before VM halts.

```

- finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다
	- 잡지 못한 예외 때문에 해당 객체는 마무리가 덜 된 상태로 남을 수 있다
- cleaner를 사용하는 라이브러리는 자신의 스레드를 통제하기 때문에 이러한 문제가 발생하지 않는다

### finalizer와 cleaner는 심각한 성능 문제를 동반한다
- finalizer는 가비지 컬렉터의 효율을 떨어뜨린다
- cleaner도 클래스의 모둔 인스턴스를 수거하는 형태로 사용하면 성능은 비슷하다

### finalizer와 cleaner는 심각한 보안 문제를 일으킬 수 있다
- finalizer와 cleaner는 finalizer 공격에 노출될 수 있다
- 생성자나 직렬화 과정에서 예외가 발생하면 이 생성되다 만 객체의 악의적인 하위 클래스의 finalizer가 수행될 수 있게 된다
- finalizer는 정적 필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있다
- 객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만 finalizer가 있다면 그렇지도 않다
	- final 클래스를 만드면 하위 클래스를 생성할 수 없으므로 해결할 수 있다
	- final 클래스가 아니라면  아무일도 하지 않는 finalize 메서드를 만들고 final로 선언해라

### finalizer와 cleaner 대신 AutoCloseable 사용
- finalizer와 cleaner의 대안은 AutoCloseable을 구현해주고 클라이언트에서 인스턴스를 다 쓰고나면 close를 호출하는 것이다
- 각 인스턴스는 자신이 닫혔는지 추적하는 것이 좋다
	- close 메서드에서 이 객체는 더이상 유효하지 않음을 필드로 기록한다
	- 그리고 다른 메서드느 이 필드를 검사해서 객체가 닫힌 후 불렸다면 IllegalStateException을 던진다

### finalizer와 cleaner의 적절한 사용
- 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할
	- `FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor`
- 네이티브 피어와 연결된 객체에서 사용
	- 네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다
	- 자바의 가비지 컬렉터는 네이티브 피어의 존재를 알지 못해 finalizer와 cleaner가 나서서 하기에 적당한 작업이다
	- 단, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을때만 해당한다 