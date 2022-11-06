# 갱신이 초래하는 트레이드오프
---
- Orders, OrderReceipts 두 개의 테이블이 있다
- Orders 테이블의 레코드 하나는 하나의 주문에 대응하고, OrderReceipts 테이블의 레코드 하나는 하나의 주문된 제품에 대응한다
- Orders 테이블과 OrderReceipts 테이블은 일대다 관계를 가진다
- 주문일(order_date)와 배송 예정일(delivery_date) 사이에 3일 이상의 차이가 있으면 배송이 지연된다는 알림을 보내고 싶다
- 어떤 주문 번호가 이에 해당될지 해결하라

**Orders**
<img width="555" alt="image" src="https://user-images.githubusercontent.com/60502370/200154898-bd77f0ee-e3e9-45ec-8d03-f94c7b816e30.png">

**OrderReceipts**
<img width="598" alt="image" src="https://user-images.githubusercontent.com/60502370/200154916-6dc1f9e8-2b20-452e-82ed-7d3863cefd32.png">

## 1. SQL을 사용하는 방법
```SQL
SELECT o.order_id,
	   o.order_name,
	   oc.delivery_date - o.order_date
FROM Orders o
	INNER JOIN OrderReceipts oc ON o.order_id = oc.order_id
WHERE oc.delivery_date - o.order_date >= 3
;
```

- 만약 주문번호별 최대 지연일을 알고 싶다면 주문번호를 집약한다
- 주문번호와 주문자 명의가 일대일 대응한다는 것이 확실하다면 주문자 이름에도 함수를 적용해서 결과를 포함할 수도 있다

```SQL
SELECT o.order_id,
	   MAX(o.order_name),
	   MAX(oc.delivery_date - o.order_date)
FROM Orders o
	INNER JOIN OrderReceipts oc ON o.order_id = oc.order_id
WHERE oc.delivery_date - o.order_date >= 3
GROUP BY o.order_id
;
```
- `O.order_name`에 MAX를 사용하는 것은 최대값을 구하는 것이 아니라 집약 대상이 아니므로 집약 함수를 사용할 수 밖에 없다
- `order_id`와 `order_name`이 일대일 대응하는 것이 확실하다면 `GROUP BY o.order_id, o.order_name` 처럼 작성하여 주문자 명의에 적용된 집약 함수를 제거할 수도 있다

## 2. 모델 갱신을 사용하는 방법
- 해당 SQL을 사용하는 것이 최적의 해답인지는 의문의 여지가있다
- 더 나은 SQL을 작성할 수 있다는 뜻이 아니라 SQL에 의지하지 않고도 해결 방법이 있을 수 있다는 것이다
- 배송이 늦어질 가능성이 있는 주문의 레코드에 대해 플래그 필드를 Orders 테이블에 추가하면, 검색 쿼리는 해당 플래그만을 조건으로 삼으므로 굉장히 간단해진다
