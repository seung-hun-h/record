# 시야 협착: 관련 문제
---
- Orders, OrderReceipts 테이블을 다시 사용해서 주문 번호별 주문 수량을 알아보려고 한다
- 결과에 포함되어야 하는 필드는 다음과 같다
	- 주문번호
	- 주문자 이름
	- 주문일
	- 상품 수

## 1. 다시 SQL을 사용한다면

**집약 함수 사용**
```SQL
SELECT o.order_id,
	   MAX(o.order_name),
	   MAX(o.order_date),
	   COUNT(1) AS item_count
FROM Orders o
	INNER JOIN OrderReceipts orc ON o.order_id = orc.order_id
GROUP BY o.order_id
;
```

**윈도우 함수 사용**
```SQL
SELECT o.order_id,
	   o.order_name,
	   o.order_date,
	   COUNT(1) OVER(PARTITION BY o.order_id) AS item_count
FROM Orders o
	INNER JOIN OrderReceipts orc ON o.order_id = orc.order_id
;
```

- 집약을 사용한 쿼리와 윈도우 함수를 사용한 쿼리 모두 결합과 집약을 수행하므로 실행 비용자체는 비슷하다

**집약 함수 사용 실행계획**
![image](https://user-images.githubusercontent.com/60502370/200720146-14f57956-0148-4dc8-8fad-8ae53dcf8348.png)


**윈도우 함수 사용 실행계획**
![image](https://user-images.githubusercontent.com/60502370/200720186-2d5fb0ca-b0c7-40ba-876c-481937c3458c.png)

- 무엇이 더 좋은 코드인지는 본인이 판단해야 한다
- 윈도우 함수 장점
	- 가독성이 좋다
	- 주문 번호가 아닌 상품 별로 결과를 출력하고 싶을 때도 쉽게 대응할 수 있다

## 2. 다시 모델 갱신을 사용한다면
- Orders 테이블에 '상품 수'라는 필드를 추가할 수도 있다
- 이러한 방법의 주의점은 나중에 주문을 변경ㅎ나다면 상품 수도 함께 변경해야 한다는 것이다
- 따라서 동기/비동기 문제를 생각해야 한다