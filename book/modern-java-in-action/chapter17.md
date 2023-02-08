# 17. 리액티브 프로그래밍
오늘날에는 세 가지 이유로 상황이 변하고 있다
- 빅데이터: 보통 빅데이터는 페타바이트 단위로 구성되며 매일 증가한다
- 다양한 환경: 모바일 디바이스에서 수천 개의 멀티 코어 프로세서로 실행되는 클라우드 기반 클러스터에 이르기까지 다양한 환경에 애플리케이션이 배포된다
- 사용 패턴: 사용자는 1년 내내 항상 서비스를 이용할 수 있으며 밀리초 단위의 응답 시간을 기대한다

리액티브 프로그래밍에서는 다양한 시스템과 소스에서 들어오는 데이터 항목 스트림을 비동기적으로 처리하고 합쳐서 위 요구 사항을 만족한다. 이러한 패러다임에 맞게 설계된 애플리케이션은 사용자에게 **높은 응답성**을 제공한다.

한 컴포넌트나 애플리케이션뿐 아니라 전체의 리액티브 시스템을 구성하는 여러 컴포넌트를 조절하는 데도 리액티브 기법을 사용할 수 있다. 이런 시스템에서는 고장, 정전 같은 상태에 대처할 뿐 아니라 다양한 네트워크 상태에서 메시지를 교환하고 전달할 수 있으며 무거운 작업을 하고 있는 상황에서도 가용성을 제공한다.

## 17.1 리액티브 매니패스토
- 반응성(Responsive)
	- 리액티브 시스템은 일정하고 **예상할 수 있는 반응 시간을 제공해 사용자가 기대치를 가질 수 있다**
	- 기대치를 통해 사용자의 확신이 증가하면서 사용할 수 있는 애플리케이션이라는 확인을 제공할 수 있다
- 회복성(Resilient)
	- 장애가 발생해도 시스템은 반응해야 한다
	- 컴포넌트 실행 복제, 컴포넌트간 시간과 공간 분리, 작업의 비동기적 위임 등 회복성을 달성할 수 있는 기법을 제공한다
- 탄력성(Elastic)
	- 무거운 작업 부하가 발생하면 자동으로 관련 컴포넌트에 할당된 자원의 수를 늘린다
- 메시지 주도(Message-driven)
	- 회복성과 탄력성을 지원하려면 약한 결합, 고립, 위치 투명성 등을 지원할 수 있도록 시스템을 구성하는 컴포넌트의 경계를 명확하게 정의해야 한다
	- 비동기 메시지를 전달해 컴포넌트간 통신이 이루어진다

### 17.1.1 애플리케이션 수준의 리액티브
- 애플리케이션 수준 컴포넌트의 리액티브 프로그래밍의 주요 기능은 비동기로 작업을 수행할 수 있다는 점이다
	- CPU 사용률을 극대화 할 수  있다
- 이러한 목표를 달성할 수 있도록 리액티브 프레임 워크와 라이브러리는 스레드를 퓨처, 액터, 일련의 콜백을 발생시키는 이벤트 루프 등과 공유하고 처리할 이벤트를 변환하고 관리한다
- 개발자는 이러한 기술을 사용해 동시, 비동기 애플리케이션 구현의 추상 수준을 높여 저 수준의 멀티스레드 문제를 직접 처리할 필요가 없다
- 리액티브 시스템을 만드려면 훌륭하게 설계된 리액티브 애플리케이션 집합이 서로 잘 조화를 이루게 만들어야 한다

### 17.1.2 시스템 수준의 리액티브
- 리액티브 시스템
	- 여러 애플리케이션이 한 개의 일관적인, 회복할 수 있는 플랫폼을 구성할 수 있게함
	- 한 애플리케이션이 실패하더라도 전체 시스템은 계속 운영될 수 있도록 함
	- 애플리케이션을 조립하고 상호 소통을 조절한다
	- 주요 속성으로는 메시지 주도가 있다
- 리액티브 애플리케이션
	- 짧은 시간동안만 유지되는 데이터 스트림에 기반한 연산을 수행
	- 이벤트 주도로 분류
- 메시지는 정의된 목적지 하나를 향하는 반면, 이벤트틑 관련 이벤트를 관찰하도록 등록한 컴포넌트가 수신한다

- 회복성의 핵심은 고립과 비결함
	- 리액티브 시스템에서는 수신자와 발신자가 메시지와 결합되지 않도록 메시지를 비동기로 처리해야 한다
	- 리액티브 시스템에서는 각 컴포넌트를 완전히 고립하여 시스템이 장애와 높은 부하에서도 반응성을 유지할 수 있다
- 탄력성의 핵심은 위치 투명성
	- 모든 컴포넌트가 수신자의 위치에 상관없이 다른 모든 서비스와 통신할 수 있음을 의미한다
	- 시스템 복제 가능, 동적 확장 가능

## 17.2 리액티브 스트림과 플로 API
- 리액티브 프로그래밍은 리액티브 스트림을 사용하는 프로그래밍이다
- 리액티브 스트림은 잠재적으로 무한의 비동기 데이터를 순서대로 그리고 블록하지 않는 역압력을 전제해 처리하는 표준기술이다
- 역압력은 발행자가 이벤트를 제공하는 속도보다 구독자가 소비하는 속도가 느릴때 문제가 발생하지 않도록 보장하는 장치이다

> 539p 2번째 문단

### 17.2.1 Flow 클래스 소개
- 자바 9에서는 리액티브 프로그래밍을 제공하는 클래스 `java.util.concurrent.Flow`를 추가했다
- 리액티브 스트림 프로젝트의 표준에 따라 프로그래밍 발행-구독 모델을 지원할 수 있도록 다음 네 개 인터페이스를 제공한다
	- Publisher
		- 항목을 발행한다
		- 역압력 기법에 의해 이벤트 제공 속도가 제한된다
		- 함수형 인터페이스다
	- Subscriber
		- 한 개씩 또는 한 번에 여러 항목을 소비한다
		- Publisher가 발행한 이벤트의 리스너로 자신을 등록할 수 있다
	- Subscription
		- Publisher와 Subscriber 사이의 제어 흐름, 역압력을 관리한다
	- Processor

```java
@FunctionalInterface
public interface Publisher<T> {
	void subscribe(Subscriber<? super T> s);
}
```

```java
public interface Subscriber<T> {
	void onSubscribe(Subscription s);
	void onNext(T t);
	void onError(Throwable t);
	void onComplete();
}
```

- 이들 이벤트는 다음 프로토콜에서 정의한 순서대로 지정된 메서드 호출을 통해 발행되어야 한다

```text
onSubscribe onNext* (onError | onComplete)?
```
- onSubscribe가 항상 처음 호출된다
- onNext가 여러번 호출될 수 있다
- onComplete 콜백을 통해 더 이상의 데이터가 없고 종료됨을 알린다
- onError 콜백을 통해 Publisher에 장애가 발생했음을 알린다

```java
public interface Subscription {
	void request(long n);
	void cancel();
}
```
- Subscriber가 Publisher에 자신을 등록할 때 Publisher는 처음으로 onSubscribe를 호출해 Subscription 객체를 전달한다
- `request`는 Publisher에게 주어진 개수의 이벤트를 처리할 준비가 되었음을 알릴 수 있다
- `cancel`는 Publisher에게 더 이상 이벤트를 받지 않음을 통지한다

![Screen Shot 2022-08-23 at 3 39 35 PM](https://user-images.githubusercontent.com/60502370/186088306-401bfcc4-b5e0-4f0a-b65c-84858979c72f.png)

```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> { }
```
- Processor는 단지 Subscriber와 Publisher를 상속받을 뿐이다
- Processor는 리액티브 스트림에서 처리하는 이벤트의 변환 단계를 나타낸다
	- 에러를 수신하면 이로부터 회복하거나 모든 Subscriber에 에러를 전파할 수 있다
	- 마지막 Subscriber가 Subscription을 취소하면 Processor는 자신의 업스트림 Subscription도 취소함으로 취소 신호를 전파해야 한다

### 17.2.2 첫 번째 리액티브 애플리케이션 만들기
- 생략

### 17.2.3 Processor로 데이터 변환하기
- 생략

### 17.2.4 자바는 왜 플로 API 구현을 제공하지 않는가?
- 생략

## 17.3 리액티브 라이브러리 RxJava 사용하기
- RxJava는 자바로 리액티브 애플리케이션을 구현하는 데 사용하는 라이브러리다
- RxJava는 Flow.Publisher를 구현하는 두 클래스를 제공한다
	- `io.reactivex.Flowable`
	- `io.reactivex.Observable`
- Flowable은 역압력을 제공하고, Observable은 역압력을 제공하지 않는다
- RxJava는 천 개 이하의 요소를 가진 스트림이나 마우스 움직임, 터치 이벤트 등 역압력을 적용하기 힘든 GUI 이벤트나 자주 발생하지 않는 이벤트에서 대해서 역압력을 적용하지 말 것을 권장한다

### 17.3.1 Observable 만들고 사용하기
- 팩토리 메서드
	- `just()`
	- `interval()`

```java
Observable<String> strings = Observable.just("first", "second");

Observable<Long> onePerSec = Observable.interval(1, TimeUnit.SECONDS);
```

- `just()`는 한 개이 상의 요소를 이용해 Observable을 반환한다
	- Observable의 구독자는 onNext("first"), onNext("second"), onComplete의 순서로 메시지를 받는다
- `interval()`는 시간 간격을 전달 받아 Observable을 반환한다
	- 사용자와 실시간으로 상호작용하면서 지정된 속도로 이벤트를 방출하는 상황에 유용하게 사용할 수 있다
	- 0에서 시작해 1초 간격으로 long 형식의 값을 무한으로 증가시키며 값을 방출한다

- Observable은 Flow API의 Publisher 역할을 하고, Observer는 Subscriber 역할을 한다

```java
public interface Observer<T> {
	void onSubscribe(Disposable d);
	void onNext(T t);
	void onError(Throwable t);
	void onComplete();
}
```

- RxJava는 자바 9 네이티브 플로 API보다 유연하다
	- 다른 세 메서드는 생략하고 onNext 메서드의 시그니처에 해당하는 람다 표현식을 전달해 Observable을 구독할 수 있다
```java

@CheckReturnValue  
@SchedulerSupport(SchedulerSupport.NONE)  
@NonNull  
public final Disposable subscribe(@NonNull Consumer<? super T> onNext) {  
    return subscribe(onNext, Functions.ON_ERROR_MISSING, Functions.EMPTY_ACTION);  
}
```

- 초당 한 개씩 이벤트를 방출하며 메시지를 수신하면 Subscriber가 뉴욕의 온도를 추출해 출력한다
```java
onePerSec.subscribe(i -> System.out.println(TempInfo.fetch("New York")));
```
- 하지만 위코드를 실행하면 아무것도 출력되지 않는다
	- 이는 매 초마다 정보를 발행하는 Observable이 RxJava의 연산 스레드 풀 즉 데몬 스레드에서 실행되기 떄문이다

- sleep 메서드를 추가하는 방법도 있지만, 현재 스레드에서 콜백을 호출하는 `blockingSubscribe`를 호출하면 된다
```java
onePerSec.blockingSubscribe(i -> System.out.println(TempInfo.fetch("New York")));
```

- 하지만 예외가 발생하는 경우 위 코드의 Observer는 onError 같은 에러 관리 기능을 포함하지 않으므로 처리되지 않는 예외가 사용자에게 직접 보여진다

![image](https://user-images.githubusercontent.com/60502370/186109856-972a7ab2-9373-40b5-916f-78c9cc56e3ec.png)

- 이제 에러 처리와 함께 기능을 일반화 해야 한다
	- 온도를 직접 출력하지 않고 사용자에게 팩토리 메서드를 제공해 매 초마다 온도를 방출하는 Observable을 반환할 것이다

![image](https://user-images.githubusercontent.com/60502370/186110184-b9286e80-b983-4116-983a-7bc64355bc58.png)

- 필요한 이벤트를 정송하는 ObservableEmitter를 소비하는 함수로 Observable을 만들어 반환했다
- RxJava의 ObservableEmitter 인터페이스는 RxJava의 기본 Emmiter(onSubscribe 메서드가 빠진 Observer와 같음)를 상속한다
- 아래 코드처럼 getTemparature 메서드가 반환하는 Observable에 가입시킬 Observer를 쉽게 완성해서 전달된 온도를 출력할 수 있다

```java
public class TempObserver implements Observer<TempInfo> {  
   @Override  
   public void onSubscribe(@NonNull Disposable d) {  
  
   }  
  
   @Override  
   public void onNext(@NonNull TempInfo tempInfo) {  
      System.out.println("tempInfo = " + tempInfo);  
   }  
  
   @Override  
   public void onError(@NonNull Throwable e) {  
      System.out.println("Got problem: " + e.getMessage());  
   }  
  
   @Override  
   public void onComplete() {  
      System.out.println("Done!");  
   }  
}
```

- 아래와 같이 실행한다

```java
public class Main {  
   public static void main(String[] args) {  
      Observable<TempInfo> observable = getTemperatures("New York");  
      observable.blockingSubscribe(new TempObserver());  
   }
}
```

### 17.3.2 Observable을 변환하고 합치기
- RxJava나 기타 리액티브 라이브러리는 자바 9 Flow API에 비해 스트림을 합치고, 만들고, 거르는 등의 풍부한 도구 상자를 제공하는 것이 장점이다
- 변환, 합치기 함수는 말로는 표현하기 어려우므로 RxJava는 마블 다이어그램을 제공해 이를 설명한다
![image](https://user-images.githubusercontent.com/60502370/186120271-14c11b03-9519-4a51-a057-1741918e8179.png)
![image](https://user-images.githubusercontent.com/60502370/186120340-44fbe320-63e3-428f-aad0-4bc0bd4788a9.png)

- map을 사용하면 Flow API의 Processor를 사용하는 것 보다 더 깔끔하게 섭씨를 화씨로 변경할 수 있다

```java
public static Observable<TempInfo> getCelsiusTemperatures(String town) {  
   return getTemperatures(town)  
      .map(tempInfo -> new TempInfo(tempInfo.getTown(), (tempInfo.getTemp() - 32) * 5 / 9));  
}
```

- 구현한 메서드를 일반화해서 사용자가 한 도시뿐 아니라 여러 도시에서 방출하는 Observable을 가질 수 있도록 해야 한다고 가정한다
- merge 함수를 이용해 하나로 합쳐서 마지막 요구사항을 구현할 수 있다

```java
public static Observable<TempInfo> getCelsiusTemperatures(String... town) {  
   return Observable.merge(Arrays.stream(town)  
      .map(Main::getTemperatures)  
      .collect(Collectors.toList()));  
}
```
