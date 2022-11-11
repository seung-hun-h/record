# 인덱스로 성능 향상이 어려운 경우
---
- 테이블의 정의와 SQL만 봐서 인덱스를 설계할 수는 없다
- 적절한 인덱스를 작성하려면, SQL의 검색 조건과 결합 조건을 바탕으로 데이터를 효율적으로 압출할 수 있는 조건을 찾아야 한다
	- 이를 위해서는 SQL 구문과 검색 키 필드의 카디널리티를 알아야 한다
	- 그 결과 운좋게 데이터를 압축할 좋은 조건을 발견하게 되면 인덱스를 생성할 수 있게된다

**예시 테이블: Orders**
```SQL
CREATE TABLE Orders
(
	order_id CHAR(8) NOT NULL,
	shop_id CHAR(4) NOT NULL,
	shop_name VARCHAR(256) NOT NULL,
	receive_date DATE NOT NULL,
	process_flg CHAR(1) NOT NULL,
	CONSTRAINT pk_Orders PRIMARY KEY(order_id)
);
```

## 1.  압축 조건이 존재하지 않음
```SQL
SELECT order_id, receive_date
FROM Orders;
```
- 레코드를 압축하는 WHERE 구가 없어 인덱스로 작성할만한 필드도 존재하지 않는다
- 테이블 풀 스캔이 일어날 것이다

## 2. 레코드를 제대로 압축하지 못하는 경우
```SQL
SELECT order_id, receive_date
FROM Orders
WHERE process_flg = '5'
;
```
- 인덱스를 만들어도 선택률이 매우 높다면 테이블 풀 스캔을 할 때보다 오히려 더 느려진다
- 인덱스가 제대로 동작하려면 '레코드를 크게 압축할 수 있는 검색 조건'이 있어야 한다

### 입력 매개변수에 따라 선택률이 변동하는 경우 - 1
- 기간의 범위 검색과 같은 경우가 있다
```SQL
SELECT order_id
FROM Orders
WHERE receive_date BETWEEN :start_date AND :end_date;
```
- start_date와 end_date에 따라서 선택률이 변동적이다

### 입력 매개변수에 따라 선택률이 변동하는 경우 - 2
- 점포 별로 주문 수를 세는 경우도 있다
```SQL
SELECT COUNT(*)
FROM Orders
WHERE shop_id = :sid;
```
- 어떤 점포는 주문 수가 100 건일 수 있고, 어떤 점포는 10만 건일 수 있다
- 점포에 따라 선택률이 높다면 오히려 테이블 풀 스캔이 더 나은 경우가 있을 수 있다
- 이때 shop_id 필드에 인덱스가 존재한다고 가정하면 선택률이 높은 점포인 경우 오히려 성능 악화를 일으킬 수 있다

## 3. 인덱스를 사용하지 않는 검색 조건
### 중간 일치, 후방 일치의 LIKE 연산자
```SQL
SELECT order_id
FROM Orders
WHERE shop_name LIKE '%대공원%'
;
```

- LIKE의 경우 전방일치에만 인덱스를 사용할 수 있다
- 예시처럼 중간 일치나 후방일치는 인덱스를 사용할 수 없다

### 색인 필드로 연산을 하는 경우
```SQL
SELECT *
FROM SomeTable
WHERE col * 1.1 > 100;
```
- 색인 필드에 연산을 적용하는 경우 인덱스를 사용할 수 없다
- 그리고 색인 필드에 함수를 적용하는 경우에도 인덱스를 사용할 수 없다

```SQL
SELECT *
FROM SomeTable
WHERE col > 100 / 1.1;
```
- 이렇게 변경하자

### IS NULL을 사용하는 경우
- NULL과 관련한 검색 조건에서 인덱스가 사용되지 않는 것은 일반적으로 색인 필드의 데이터에 NULL이 존재하지 않기 때문이다

```SQL
SELECT *
FROM SomeTable
WHERE col IS NULL;
```

### 부정형을 사용하는 경우
- 부정형 (<>, !=, NOT IN)은 인덱스를 사용할 수 없다