# 16. CompletableFuture: 안정적 비동기 프로그래밍
## 16.1 Future의 단순 활용
- 자바 5부터는 `Future` 인터페이스를 제공한다
  - 비동기 모델링 하는데 사용할 수 있다
  - 계산이 끝났을 때 결과에 접근할 수 있는 참조를 제공한다(`get()`)
  - 시간이 걸릴 수 있는 작업을 `Future` 내부로 설정하면 호출자 스레드가 결과를 기다리는 동안 다른 유용한 작업을 수행할 수 있다

![image](https://user-images.githubusercontent.com/60502370/183290831-98c01ba2-7342-4cf1-afc7-416c6076e58e.png)

- 위 코드에서는 오래 걸리는 작업이 영원히 끝나지 않는 경우에 대비해 타임아웃을 설정하였다

### 16.1.1 Future 제한
- `Future` 인터페이스는 계산이 끝났는지 확인 할 수 있는 메서드, 계산이 끝나길 기다리는 메서드, 결과 회수 메서드 등을 제공한다
- 이들 메서드 만으로는 간결한 동시 실행 코드를 구현하기 충분하지 않다
  - 여러 `Future`의 결과가 있을 때 이들의 의존성을 표현하기 어렵다
  - 'A의 연산이 끝난 후 B 연산을 수행하고 다른 질의 결과와 B의 결과를 조합하라' 와 같은 요구 사항을 구현하기 어렵다
- 다음과 같은 선언형 기능이 있다면 동시 실행 코드를 구현하기 유용할 것이다
  - 두 개의 비동기 계산 결과를 합친다. 두 개의 연산은 독립적일 수도 있고, 순서에 의존할 수 있다
  - `Future` 집합이 실행하는 모든 태스크의 완료를 기다린다
  - `Future` 집합에서 가장 빨리 완료되는 태스크를 기다렸다가 결과를 얻는다
  - 프로그램적으로 `Future`를 완료시킨다(비동기 동작에 수동으로 결과 제공)(?)
  - `Future` 완료 동작에 반응한다

- `CompletableFuture`는 이러한 기능을 선언형으로 사용할 수 있도록 제공한다

### 16.1.2 CompletableFuture로 비동기 애플리케이션 만들기
- 내용 생략

**동기 API와 비동기 API**
- 동기 API는 메서드를 호출한 다음에 메서드가 계산을 완료할 때까지 기다렸다가 메서드가 반환되면 호출자는 반환된 값으로 계속 다른 동작을 수행한다
  - 호출자와 피호출자가 다른 스레드에서 실행되더라도 호출자는 피호출자의 동작 완료를 기다렸을 것이다
  - 이처럼 동기 API를 사용하는 상황을 블록 호출이라 한다
- 비동기 API는 메서드가 즉시 반환되며 끝내지 못한 나머지 작업을 호출자 스레드와 동기적으로 실행될 수 있도록 다른 스레드에 할당한다
  - 비동기 API를 사용하는 상황을 비블록 호출이라고 한다

## 16.2 비동기 API 구현
- 최저가 검색 애플리케이션을 구현한다
```java
public class Shop {
  public double getPrice(String product) {
    // 구현 해야 함
  }
}
```
- `getPrice`는 상점의 데이터베이스를 이요해서 가격 정보를 얻는 동시에 다른 외부 서비스도 접근한다

```java
public double getPrice(String product) {
  return calculatePrice(product);
}

private double caculatePrice(String product) {
  delay();
  return random.nextDouble() * product.charAt(0) + product.charAt(1);
}

public void delay() {
  try {
    Thread.sleep(1000L);
  } catch (InterruptedException e) {
    throw new RuntimeException(e);
  }
}
```

- `delay`는 오래 걸리는 작업을 흉내내기 위한 메서드다
- `calculate`는 임의의 계산값을 반환한다
- 최저가 검색 애플리케이션에서 위 메서드를 사용해서 네트워크 상의 모든 온라인 상점의 가격을 검색해야 하므로 블록 동작은 바람직하지 않다
- 예제애서 편의상 사용자가 편리하게 이용할 수 있도록 비동기 API를 만들기로 결정했다고 가정한다

### 16.2.1 동기 메서드를 비동기 메서드로 변환
- 동기 `getPrice`를 비동기 `getPriceAsync`로 이름과 반환 값을 변경한다

```java
public Future<Double> getPriceAsync(String product) {
  CompletableFuture<Double> futurePrice = new CompletableFuture<>(); // 계산 결과를 포함할 CompletableFuture
  new Thread(() -> {
    double price = calculatePrice(product); // 다른 스레드에서 비동기적으로 계산 수행
    futurePrice.complete(price); // 오랜 시간이 걸리는 계산이 완료되면 Future에 값을 설정
  }).start();
  return futurePrice; // 계산 결과가 완료되길 기다리지 않고 Future 반환
}
```

- `getPriceAsync` 활용

```java
Shop shop = new Shop("BestShop");
long start = System.nanoTime();
Future<Double> futurePrice = shop.getPriceAsync("my favorite product");
long invocationTime = ((System.nanoTime() - start) / 1_000_000);

doSomeThingElse();

try {
  double price = futurePrice.get(); // 가격 정보가 있으면 Future에서 가격 정보를 읽고 없으면 받을 때까지 블록
  
} catch (Exception e) {
  throw new RuntimeException(e);
}

long retrievalTime = ((System.nanoTime() - start) / 1_000_000);
```

### 16.2.2 에러 처리 방법
- 예외가 발생하면 해당 스레드에만 영향을 미친다
- 이로인해 클라이언트는 `get`  메서드가 반환될 때까지 영원히 기다리게 될 수 있다

- 클라이언트는 타임아웃 값을 받는 `get` 메서드의 오버로드 버전을 만들어 문제를 해결할 수 있다
  - 블록 문제가 발생할 수 있는 상황에서는 타임아웃을 활용하는 것이 좋다
  - 그리고 `CompletableFuture` 내부에서 발생한 예외를 클라이언트로 전달하기 위해 `completeExceptionally` 메서드를 이용한다

```java
public Future<Double> getPriceAsync(String product) {
  CompletableFuture<Double> futurePrice = new CompletableFuture<>(); // 계산 결과를 포함할 CompletableFuture
  new Thread(() -> {
    try {
      double price = calculatePrice(product); // 다른 스레드에서 비동기적으로 계산 수행 
      futurePrice.complete(price); // 오랜 시간이 걸리는 계산이 완료되면 Future에 값을 설정
    } catch (Exception ex) {
      futurePrice.completeExceptionally(ex); // 도중에 문제가 발생하면 발생한 에러를 포함시켜 Future를 종료한다
    }
    
  }).start();
  return futurePrice; // 계산 결과가 완료되길 기다리지 않고 Future 반환
}
```

**팩토리 메서드로 supplyAsync로 CompletableFuture 만들기**
- `CompletableFuture`의 팩토리 메서드를 사용하면 좀 더 간단히 만들 수 있다

```java
public Future<Double> getPriceAsync(String product) {
  return CompletableFuture.supplyAsync(() -> calculatePrice(product));
}
```

- `supplyAsync`는 `Supplier`를 인수로 받아 `CompletableFuture`를 반환한다
- `CompletableFuture`는 `Supplier`를 실행해서 비동기적으로 결과를 생성한다
- `ForkJoinPool`의 Executor 중 하나가 `Supplier`를 실행할 것이다


## 16.3 비블록 코드 만들기
- 다음과 같은 상점 리스트가 있다고 가정한다

```java
List<Shop> shops = Arrays.asList( ... )
```
- 제품명을 입력하면 상점 이름과 제품 가격 문자열 정보를 포함하는 리스트를 반환하는 메서드를 구현한다

```java
public List<String> findPrices(String product) {
  return shops.stream()
            .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
            .collect(toList());
}
```
- 스트림을 사용하면 간단하게 구현할 수 있다
- 하지만 아직 부족하다. 성능을 개선해보자

### 16.3.1 병렬 스트림으로 요청 병렬화하기

```java
public List<String> findPrices(String product) {
  return shops.parallelStream()
            .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
            .collect(toList());
}
```

### 16.3.2 CompletableFuture로 비동기 호출 구현하기
```java
public List<String> findPrices(String product) {
  return shops.parallelStream()
            .map(shop -> CompletableFuture.supplyAsync(() -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
            .collect(toList());
}
```
- `findPrices`의 반환 형식은 리스트이므로 모든 `CompletableFuture`의 동작이 완료되고 결과를 추출한 다음에 결과를 반환해야 한다
- 그러나 위 코드로는 `List<CompletableFuture<String>>`을 반환한다

- `CompletableFuture`의 `join`을 사용해서 모든 동작이 끝나기를 기다리도록 코드를 수정한다

```java
public List<String> findPrices(String product) {
  List<CompletableFuture<String>> priceFutures = shops.parallelStream()
                          .map(shop -> CompletableFuture.supplyAsync(() -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
                          .collect(toList());

  return priceFutures.stream()
                    .map(CompletableFuture::join)
                    .collect(toList());
}
```
- 두 개의 스트림 파이프라인을 사용해서 처리했다
- 스트림 연산은 게으른 특성이 있어서 하나의 파이프 라인으로 연산을 처리했다면 모든 가격 정보 요청 동작이 동기적, 순차적으로 이루어지는 결과가 된다

- 실행해보면 기존 코드보다는 결과가 좋지만, 병렬로 실행헀을 때 보다는 성능이 떨어진다

### 16.3.3 더 확장성이 좋은 해결 방법
- 상점을 추가해서 실행하면 두 가지 방법이 비슷한 결과를 보인다
- `CompletableFuture`는 병렬 스트림 버전에 비해 작업에 이용할 수 있는 다양한 Executor를 지정할 수 있다는 장점이있다
- 따라서 Executor로 스레드 풀의 크기를 조절하는 등 애필리케이션에 맞는 최적화된 설정을 만들 수 있다

### 커스텀 Executor 사용하기
- 전략
- 애플리케이션에 맞는 Exeuctor를 만들어 `CompletableFuture`를 활용하는 것이 바람직하다

 **스트림 병렬화와 CompletableFuture 병렬화**
 - 지금까지 컬렉션 계산을 병렬화하는 두 가지 방법을 살펴봤다
 - 하나는 병렬 스트림으로 변환해서 컬렉션을 처리하는 방법이고
 - 하나는 컬렉션을 반복하면서 `CompletableFuture` 내부 연산으로 만드는 것이다
   - 전체적인 계산이 블록되지 않도록 스레드 풀의 크기를 조절할 수 있다

- 다음을 참고하면 어떤 병렬화 기법을 사용할 것인지 선택하는데 도움이 된다
  - I/O가 포함되지 않는 계산 중심의 동작을 실행할 때는 스트림 인터페이스가 가장 구현하기 간단하며 효율적일 수 있다
  - 작업이 I/O를 기다리는 작업을 병렬로 실행할 때는 CompletableFuture가 더 많은 유연성을 제공하면 대기/계산의 비율에 적합한 스레드 수를 설정할 수 있다. 특히 스트림의 게으른 특성때문에 스트림에서 I/를 실제로 언제 처리할 지 예측하기 어려운 문제도 있다

## 16.4 비동기 작업 파이프라인 만들기
- 모든 상점이 하나의 할인 서비스를 사용하기로 했다고 가정
- 할인 서비스에는 서로 다른 할인율을 제공하는 다섯 가지 코드를 제공한다

```java
public class Discount {
	public enum Code {
		NONE(0), SILVER(5), GOLD(10), PLATINUM(15), DIAMOND(20);

		private final int percentage;

		Code(int percentage) {
			this.percentage = percentage;
		}
	}
}
```

- 그리고 상점에서 `getPrice`의 결과 형식도 변경한다
	- `ShopName:price:DiscodeCode` 형식의 문자열을 반환한다

```java
public String getPrice(String product) {
	double price = calculatePrice(product);
	Discount.Code code = Discount.Code.values()[random.nextInt(Discount.Code.values().lenth)];
	
	return String.format("%s:%.2f:%s", name, price, code);
}

private double calculatePrice(String product) {
	delay();
	return random.nextDouble() * product.charAt(0) + product.charAt(1);
}
```

### 16.4.1 할인 서비스 구현
- 상점에서 제공한 문자열 파싱은 다음처럼 `Quote` 클래스로 캡슐화할 수 있다

```java
public class Quote {
	private final String shopName;
	private final double price;
	private final Discount.Code discountCode;

	public Quote(String shopName, double price, Discount.Code code) {
		this.shopName = shopName;
		this.price = price;
		this.discountCode = code;
	}
	
	public static Quote parse(String s) {
		String[] split = s.split(":");
		String shopName = split[0];
		double price = Double.parseDouble(split[1]);
		Discount.Code discountCode = Discount.Code.valueOf(split[2]);
		return new Quote(shopName, price, discountCode);
	}

	// getter, setter
}
```
- `Discount` 서비스에서는 `Quote` 객체를 인수로 받아 할인된 가격 문자열을 반환하는 `applyDiscount` 메서드도 제공한다

```java
public class Discount {
	public enum Code {
		// 생략
	}
	
	public static String applyDiscount(Quote quote) {
		return quote.getShopName() + " price is " + Discount.apply(quote.getPrice(), quote.getDiscountCode());
	}

	private static double apply(double price, Code code) {
		delay();
		return format(price * (100 - code.percentage / 100);
	}
}
```

### 16.4.2 할인 서비스 사용
- `Discount`는 원격 서비스이므로 1초의 지연을 추가한다
- 일단 순차적 동기 방식으로 `findPrices`메서드를 구현한다

```java
public List<String> findPrices(String product) {
	return shops.stream()
				.map(shop -> shop.getPrice(product))
				.map(Quote::parse)
				.map(Discount::applyDiscount)
				.collect(toList());
}
```

- 순차적으로 상점의 가격 정보를 요청하고 할인 코드를 적용하기 때문에 성능을 기대하기 어렵다
- 병렬 스트림을 사용하면 쉽게 성능을 개선할 수 있지만, 스레드 풀의 사이즈가 고정되어 있기 때문에 상점 수가 늘어났을 때처럼 검색 대상이 확장되었을 때 유연하게 대응할 수 없다
- `CompletableFuture`에서 수행하는 태스크를 설정할 수 있는 커스텀 `Executor`를 정의함으로써 CPU 사용을 극대화 할 수 있다

### 16.4.3 동기 작업과 비동기 작업 조합하기
- `CompletableFuture`에서 제공하는 기능으로 `findPrices`메서드를 비동기적으로 재구현한 코드

```java
public List<String> findPrices(String product) {
	List<CompletableFuture<String>> priceFutures = 
					shops.stream()
							.map(shop -> CompletableFuture.supplyAsync(() -> shop.getPrice(product), executor))
							.map(future -> future.thenApply(Quote::parse))
							.map(future -> future.thenCompose(quote -> CompletableFuture.supplyAsync(() -> Discount.applyDiscount(quote), executor)))
							.collect(toList());

	return priceFutures.stream()
						.map(CompletableFuture::join)
						.collec(toList());
}
```

![Screen Shot 2022-08-21 at 10 45 57 AM](https://user-images.githubusercontent.com/60502370/185771860-53c4fd32-295f-4825-8ae1-0419f80abedc.png)

**가격 정보 얻기**
- 팩토리 메서드 `supplyAsync`에 람다 표현식을 전달해서 비동기적으로 상점에서 정보를 조회했다
- 첫 번쨰 변환의 결과는 `Stream<CompletableFuture<String>>` 이다
- 각 `CompletableFuture` 작업이 끝났을 때 해당 상점에서 반환하는 문자열 정보를 포함한다

**Quote 파싱하기**
- 두 번째 변환 과정에서는 첫 번째 결과 문자열을 `Quote`로 변환한다
- 파싱 동작에는 원격 서비스나 I/O가 없으므로 지연없이 동작한다
- 따라서 `thenApply` 메서드를 호출한 다음에 문자열을 `Quote`인스턴스로 변환하는 `Function`으로 전달한다
	- `thenApply` 메서드는 `CompletableFuture`가 끝날 때까지 블록하지 않는다

**CompletableFuture를 조합해서 할안된 가격 계산하기**
- 세 번째 `map`연산에서는 상점에서 받은 할인전 가격에 원격 `Discount`서비스에서 제공하는 할인율을 적용해야 한다
- 원격 실행이 포함되므로 동기적으로 작업을 수행해야 한다(`thenCompose`가 동기적임)
- 람다 표현식으로 이 동작을 팩토리 메서드 `supplyAsync`에 전달할 수 있다
	- 그러면 다른 `CompletableFuture`가 반환된다
- 결국 두 가지  `CompletableFuture`로 이루어진 연쇄적으로 수행되는 두 개의 비동기 동작을 만들 수 있다
	- 상점에서 가격 정보를 얻어와서 `Quote`로 변환하기
	- 변환된 `Quote`를 `Discount` 서비스로 전달해서 할인된 최종가격 획득하기

- 세 개의 `map` 연산 결과 스트림 요소를 리스트로 수집하면 `List<CompletableFuture<String>>` 형식으 자료를 얻을 수 있다
- 마지막으로 `CompletableFuture`가 완료되기를 기다렸다가 `join`으로 값을 추출할 수 있다

### 16.4.4 독립 CompletableFuture와 비독립 CompletableFuture 합치기
- 종종 독립적으로 실행된 두 개의 `CompletableFuture` 결과를 합쳐야 하는 상황이 종종 발생한다
- 이런 상황에서는 `thenCombine` 메서드를 사용한다
	- `BiFunction`을 두 번째 인수로 받는다
	- 이는 두 개의 결과를 어떻게 합칠 지 정의한다
	- Async 버전이 존재한다
	- 