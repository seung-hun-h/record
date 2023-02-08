# 8강 UNION을 사용한 쓸데없이 긴 표현
---
- `UNION`을 사용한 조건 분기는 SQL 초보자가 좋아하는 기술이다
- WHERE 구만 조금씩 다른 여러 개의 SELECT 구문을 합쳐서, 복수의 조건에 일치하는 하나의 결과 집합을 얻고 싶을 때 사용한다
- 하지만 이런 방식은 내부적으로 여러 개의 SELECT 구문을 실행 하는 실행 계획으로 해석된다
	- 테이블에 접근하는 횟수가 많아져 I/O 비용이 크게 늘어난다

## 1. UNION을 사용한 조건 분기와 관련된 간단한 예제
- 상품을 관리하는 Items 테이블이 존재한다
- 각각 상품에 대해서 세금이 포함된 금액과 세금이 포함되지 않는 금액이있다
- 2001년까지는 세금이 포함되지 않는 금액을, 2002년 부터는 세금이 포함된 금액을 price로 표시한다

**Items 테이블**
| item_id | year | item_name | price_tax_ex | price_tax_in |
| ------- | ---- | --------- | ------------ | ------------ |
| 100     | 2000 | 머그컵    | 500          | 525          |
| ...     | ...  | ...       | ...          | ...          |

**UNION을 사용한 조건 분기**

```SQL
SELECT item_name, year, price_tax_ex AS price
FROM Items
WHERE year <= 2001
UNION ALL
SELECT item_name, year, price_tax_in AS price
FROM Items
WHERE year >= 2002
;
```

- 조건이 배타적이므로 중복된 레코드가 발생하지 않는다
- 쓸데없이 정렬 등의 처리를 하지 않아도 되므로 UNION ALL을 사용했다(?)

### UNION을 사용했을 때 실행계획 문제
- UNION 쿼리를 사용하면 Items 테이블에 2회 접근한다
- 그리고 그때마다 TABLE ACCESS FULL이 발생하므로 읽어드리는 시간이 테이블의 크기에 따라 선형으로 증가한다
- 데이터 캐시에 테이블의 데이터가 있으면 증상이 완화되겠지만, 테이블의 크기가 커지면 캐시 히트율이 낮아져 그것도 기대하기 어렵다

### 정확한 판단 없는 UNION 사용 회피
- UNION은 쉽지만 비용이 크다
- SELECT 구문 전체를 여러 번 사용해서 코드를 길게 만드는 것은 쓸데 없는 테이블 접근을 발생시키며 SQL의 성능을 나쁘게한다

## 2. WHERE 구에서 조건 분기를 하는 사람은 초보자
- "조건 분기를 WHERE 구로 하는 사람은 초보자다. 잘 하는 사람은 SELECT 구만으로 조건 분기를 한다"라는 격언이 있다
- 이 문제도 SELECT 구만으로 조건 분기를 할 수 있다

```SQL
SELECT item_name, year,
	CASE WHEN year <= 2001 THEN price_tax_ex
		 WHEN year >= 2002 THEN price_tax_in END AS price
FROM Items
WHERE year <= 2001
;
```

## 3. SELECT 구를 사용한 조건 분기의 실행 계획
- SELECT 구를 사용해 조건 분기를 하면 Items 테이블에 대한 접근히 1회로 줄었다
- SQL 구문의 성능이 좋은지 나쁜지느 반드시 실행 계획 레벨에서 판단해야 한다

---

- UNION을 사용하는 분기는 SELECT 구문을 기본 단위로 분기하고 있다
- CASE 식을 사용한 분기는 문자 그대로 식을 바탕으로하는 사고다
- '구문'에서 '식'으로 사고를 변경하는 것이 SQL을 마스터하는 열쇠 중 하나이다