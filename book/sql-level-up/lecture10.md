# 10강 그래도 UNION이 필요한 경우
---
## 1. UNION을 사용할 수 밖에 없는 경우
- 머지 대상이 되는 SELECT 구문들에서 사용하는 테이블이 다른 경우가 대표적이다

```SQL
SELECT col_1
FROM Table_A
WHERE col_2 = 'A'
UNION ALL
SELECT col_3
FROM Table_B
WHERE col_4 = 'B'
;
```

- FROM 구에서 테이블을 결합하면 CASE 식을 사용해 원하는 결과를 얻을 수도 있다
- 하지만 필요 없는 결합이 발생해서 성능적으로 악영향이 발생한다
- 따라서 실행 계획을 확인해서 어떤 것이 더 좋은 지 명확하게 확인해줘야 한다

## 2. UNION을 사용하는 것이 성능적으로 더 좋은 경우
- UNION 외 다른 방법을 사용할 수 있지만, UNION을 사용하는 편이 더 성능이 좋을 수 있는 경우가 있다
- 바로 인덱스를 사용하는 경우이다
	- UNION을 사용하면 좋은 인덱스를 사용하지만, 
	- 이외의 경우에는 테이블 풀 스캔이 발생한다면
	- UNION을 사용하는 것이 성능적으로 더 좋을 수 있다

### 예시
- 3개의 날짜 필드 date_1 ~ date_3과 그것과 짝을 이루는 플래그 필드 flg_1 ~ flg_3을 가진 테이블이 있다
- 레코드는 3개의 짝에서 하나의 짝에만 값이 있고, 다른 짝에는 값이 없다

| key | name | date_1    | flg_1 | date_2     | flg_2 | date_3 | flg_3 |
| --- | ---- | --------- | ----- | ---------- | ----- | ------ | ----- |
| 1   | a    | 2013-11-1 | T     |            |       |        |       |
| 2   | b    |           |       | 2013-11-01 | T     |        |       |
| ... | ...  | ...       | ...   | ...        | ...   | ...    | ...   |

- 특정 날짜(2013-11-01)에 대응 되는 플래그 값이 'T'인 레코드를 선택한다고 가정한다


#### UNION을 사용한 방법

```SQL
SELECT key, name,
		date_1, flg_1,
		date_2, flg_2,
		date_3, flg_3,
FROM ThreeElements
WHERE date_1 = '2013-11-01'
	AND flg_1 = 'T'
UNION
SELECT key, name,
		date_1, flg_1,
		date_2, flg_2,
		date_3, flg_3,
FROM ThreeElements
WHERE date_2 = '2013-11-01'
	AND flg_2 = 'T'
UNION
SELECT key, name,
		date_1, flg_1,
		date_2, flg_2,
		date_3, flg_3,
FROM ThreeElements
WHERE date_3 = '2013-11-01'
	AND flg_3 = 'T'
;
```

- 그리고 다음처럼 각 필드 조합에 대해 인덱스를 생성한다

```SQL
CREATE INDEX IDX_1 ON ThreeElements (date_1, flg_1);
CREATE INDEX IDX_2 ON ThreeElements (date_2, flg_2);
CREATE INDEX IDX_3 ON ThreeElements (date_3, flg_3);
```

- 실행 계획을 보면 인덱스를 타는 것을 알 수 있다
- 테이블의 레코드 수가 많고, 각각의 WHERE 구의 검색 조건에서 레코드 수를 많이 압축할수록, 테이블 풀 스캔 보다도 훨씬 빠른 접근 속도를 기대할 수 있다

#### OR을 사용한 방법
- UNION을 사용하지 않고 OR을 사용해서 해결할 수도 있다

```SQL
SELECT key, name,
		date_1, flg_1,
		date_2, flg_2,
		date_3, flg_3,
FROM ThreeElements
WHERE (date_1 = '2013-11-01' AND flg_1 = 'T')
	OR (date_2 = '2013-11-01' AND flg_2 = 'T')
	OR (date_3 = '2013-11-01' AND flg_3 = 'T')
;
```

- 하지만 WHERE 구문에서 OR을 사용하면 인덱스를 사용할 수 없다
- 따라서 테이블 풀 스캔이 발생한다
- 3회의 인덱스 스캔 VS 1회의 테이블 풀 스캔 중 어느 것이 더 빠른지 문제가 된다
	- 이는 테이블 크기와 검색 조건에 따른 선택 비율에 따라 답이 달라진다
	- 테이블이 크고, WHERE 조건으로 선택되는 레코드의 수가 충분히 작다면 UNION이 더 빠르다

#### IN을 사용한 방법
```SQL
SELECT key, name,
		date_1, flg_1,
		date_2, flg_2,
		date_3, flg_3,
FROM ThreeElements
WHERE ('2013-11-01', 'T')
		IN((date_1, flg_1),
			(date_2, flg_2),
			(date_3, flg_3))
;
```

- OR 보다 간단하고 이해하기 쉽지만 실행 계획은 OR을 사용할떄와 동일하다

#### CASE 식을 사용한 방법
```SQL
SELECT key, name,
		date_1, flg_1,
		date_2, flg_2,
		date_3, flg_3,
FROM ThreeElements
WHERE CASE WHEN date_1 = '2013-11-01' THEN flg_1
		   WHEN date_2 = '2013-11-01' THEN flg_2
		   WHEN date_3 = '2013-11-01' THEN flg_3
		   ELSE NULL END = 'T'
;
```
- CASE 식을 사용하는 방법도 OR과 IN을 사용한 방법과 실행 계획이 동일하다
