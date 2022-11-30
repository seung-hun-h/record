## 정확한 답이 필요하다면 float와 double은 피하라
---
- float과 double은 과학과 공학 계산용으로 설계되었다
- 이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 세심하게 설계되었다
- float과 double은 금융 계산처럼 정확한 결과가 필요할 때는 사용하면 안된다

### 정확한 결과를 원할때는 BigDecimal, int, long을 사용하자
- 금융 계산 처럼 정확한 값이 필요할 때는 BigDecimal, int ,long 같은 타입을 사용하자

```Java
public static void main(String[] args) {
	final BigDecimal TEN_CENTS = new BigDecimal(".10");

	int itemBoughts = 0;
	BigDecimal funds = new BigDecimal("1.00");

	for (BigDecimal price = TEN_CENTS; 
			funds.compareTo(price) >= 0; 
				price = price.add(TEN_CENTS)) {
		funds = funds.substract(price);
		itemsBought++;
	}
}
```

- 하지만 BigDecimal의 단점 두 가지가 있다
	- 기본 타입보다 사용하기 불편하다
	- 느리다
- BigDecimal의 대안으로 int, long을 사용하면 된다
	- 단, 다룰 수 있는 값의 크기가 제한되고 소수점을 직접 관리해야 한다
	- 위 경우 모든 계산을 달러 대신 센트로하면 문제가 해결된다

```Java
public static void main(String[] args) {
	int itemBoughts = 0;
	long funds = 100;

	for (int price = 10; funds >= price; price += 10) {
		funds -= price;
		itemsBoughts++;
	}
}
```

