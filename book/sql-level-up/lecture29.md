# 같은 테이블의 다른 레코드로 갱신
---
- Stocks 테이블은 주식 거래 정보를 저장하는 테이블이다
	- 종목별 거래를 행한 날짜의 주가가 등록되어있다
- 주가 테이블을 사용해 trend 필드를 연산하여 비어있는 테이블에 데이터를 INSERT 하는 문제를 살펴본다
- 이전 종가와 현재 종가를 비교해서 올랐다면 '↑', 내렸다면 '↓', 그대로라면 '→'라는 값을 저장한다

## 1. 상관 서브쿼리 사용
```SQL
INSERT INTO Stocks2
SELECT brand, sale_date, price,
	   CASE SIGN(price - (SELECT price
	   					  FROM Stocks S1
	   					  WHERE brand = Stocks.brand
	   					  AND sale_date = (SELECT MAX(sale_date)
	   									   FROM Stocks S2
	   									   WHERE brand = Stocks.brand
	   									   AND sale_date < Stocks.sale_date)))
	   	WHEN 1 THEN '↑'
	   	WHEN 0 THEN '→'
	   	WHEN -1 THEN '↓'
	   	ELSE NULL END
FROM Stocks;
```

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/199850643-51326ddc-35d5-42e5-995b-ec1f6a3e0f6d.png)
- Stocks 테이블에 대해 수차례 접근이 일어난다

## 2. 윈도우 함수 사용
```SQL
SELECT brand, sale_date, price,
	   CASE SIGN(price - MAX(price) OVER(PARTITION BY brand
	   						 			     ORDER BY sale_date
	   									 ROWS BETWEEN 1 PRECEDING
	   										      AND 1 PRECEDING))
	   	WHEN 1 THEN '↑'
	   	WHEN 0 THEN '→'
	   	WHEN -1 THEN '↓'
	   	ELSE NULL END
FROM Stocks;
```

**실행 계획**![image](https://user-images.githubusercontent.com/60502370/199850827-7f93ffbb-af4c-49f7-a357-3b74d32e1c24.png)
- Stocks 테이블에 대한 풀 스캔이 1회로 줄었다

## 3. INSERT와 UPDATE 어떤 것이 좋을까?

### INSERT SELECT의 장점
- UPDATE에 비해 성능적으로 낫다
- MySQL처럼 자기 참조를 허가하지 않는 데이터베이스에서도 INSERT SELECT를 사용할 수 있다
	- 단, 참조 대상 테이블과 갱신 대상 테이블이 서로 다른 테이블이다
### INSERT SELECT의 단점
- 같은 크기와 구조를 가진 데이터를 두 개 만들어야 한다
	- 저장소 용량이 2배 이상 소비된다
	- 저장소의 가격이 낮아진점을 고려하면 그리 큰 단점은 아니다
### 뷰로 만드는 것은 어떨까?
- Stocks2를 뷰로 만드는 것도 하나의 방법이다
- 저장소 용량을 절약할 수 있고 정보를 항상 최신으로 유지할 수있다
- 하지만 Stocks2 뷰에 접근이 발생할 때마다 복접한 연산이 수행되어 Stocks2에 접근하는 쿼리 성능이 낮아진다
	- 성능과 동기성의 트레이드 오프이다