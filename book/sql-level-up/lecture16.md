# 16강 SQL에서는 반복을 어떻게  처리할까?
## 1. 포인트는 CASE 식과 윈도우 함수
- SQL에서 반복을 대신하는 수단은 CASE 식과 윈도우 함수이다
- CASE 식은 절차 지향형 언어에서 말하는 IF-THEN-ELSE 구문에 대응하는 기능이다

```SQL
SELECT company,
	   year,
	   sale,
	   CASE SIGN(sale - MAX(sale) 
						   OVER(PARTITION BY company
							    ORDER BY year
							    ROWS BETWEEN 1 PRECEDING
									    AND 1 PRECEDING))
			WHEN 0 THEN '='
			WHEN 1 THEN '+'
			WHEN -1 THEN '-'
			ELSE NULL END as var
FROM Sales;
```

- SIGN 함수는 숫자 자료형을 매개변수로 받아 음수면 -1, 양수면 1, 0이면 0을 반환하는 함수이다
- ROWS BETWEEN 옵션은 대상 범위의 레코드를 직전 1개로 제한하는 것이다
	- ROWS BETWEEN 1 PRECEDING AND 1 PRECEDING은 '현재 레코드에서 1개 이전부터 1개 이전까지의 레코드 범위'를 나타낸다
	- 2개 전 레코드로 하고 싶으면 'ROWS BETWEEN 2 PRECEDING AND 2 PRECEDING'으로 하면된다
- 이러한 쿼리는 윈도우 함수가 보급되기 전에 사용하던 상관 서브쿼리로는 하기 힘든 것이다

**윈도우 함수로 '직전 회사명'과 '직전 매상' 검색**
```SQL
SELECT company,
	   year,
	   sale,
	   MAX(company)
		   OVER (PARTITION BY company
				 ORDER BY year
				 ROWS BETWEEN 1 PRECEDING
					  AND 1 PRECEDING) AS pre_company,
	   MAX(sale)
		   OVER (PARTITION BY company
				 ORDER BY year
				 ROWS BETWEEN 1 PRECEDING
					  AND 1 PRECEDING) AS pre_sale
FROM Sales;
```

---
#### 상관 서브쿼리를 사용한 대상 레코드 제한
- 상관 서브쿼리는 서브쿼리 내부에서 외부 쿼리와의 결합 조건을 사용하고, 해당 결합 키로 잘라진 부분 집합을 조작하는 기술이다

**상관 서브 쿼리로 '직전 회사명'과 '직전 매상' 검색**
```SQL
SELECT company,
	   year,
	   sale,
	   (SELECT company
	    FROM Sales S2
	    WHERE S1.company = S2.company
	    AND year = (SELECT MAX(year)
				    FROM Sales S3
				    WHERE S1.company = S3.company -- 상관 서브쿼리의 결합 조건
				    AND S1.year > S3.year)) AS pre_company,
	   (SELECT sale
	    FROM Sales S2
	    WHERE S1.company = S2.company
	    AND year = (SELECT MAX(year)
				    FROM Sales S3
				    WHERE S1.company = S3.company -- 상관 서브쿼리의 결합 조건
				    AND S1.year > S3.year)) AS pre_sale
FROM Sales S1;
```

---

## 2. 최대 반복 횟수가 정해진 경우
- 반복에 의존하지 않고 포장계로 문제를 해결하는 예제

### 인접한 우편 번호 찾기
- 413-0033처럼 하이픈으로 구분된 7자리 숫자를 우편번호로 사용한다
	- 왼쪽 세 자리는 지역, 오른쪽 네 자리는 해당 지역을 조금 더 자세하게 나누어 일련 번호를 붙인 것이다
	- 우편번호는 유일하다
	- 하위 자릿수가 일치할 수록 가까운 지역이다
- 주어진 우편번호와 가장 인접한 우편번호를 찾아야한다
	- 4130033이 테이블에 있으면 반환한다
	- 없으면, 413003* 이 테이블에 있으면 반환한다
	- 없으면, 41300**  이 테이블에 있으면 반환한다
	- ...
- 절차 지향형 사고 방식이라면 최대 7번 반복해 답을 도출한다. 이러한 방식은 레코드가 많아질 수록 성능이 악화된다

### 결국, 순위 붙이기 문제
- 이 문제의 포인트는 순위다

**우편번호 순위를 매기는 쿼리**
```SQL
SELECT pcode,
	   district_name,
	   CASE WHEN pcode = '4130033' THEN 0
		    WHEN pcode = '413003%' THEN 1
		    WHEN pcode = '41300%' THEN 2
		    WHEN pcode = '4130%' THEN 3
		    WHEN pcode = '413%' THEN 4
		    WHEN pcode = '41%' THEN 5
		    WHEN pcode = '4%' THEN 6
			ELSE NULL END AS rank
FROM PostalCode;
```

- 순위가 가장 높다는 것은 rank 값이 가장 낮다는 것이므로 MIN을 사용할 수 있다

```SQL
SELECT pcode,
	   district_name
FROM PostalCode
WHERE CASE WHEN pcode = '4130033' THEN 0
		   WHEN pcode = '413003%' THEN 1
		   WHEN pcode = '41300%' THEN 2
		   WHEN pcode = '4130%' THEN 3
		   WHEN pcode = '413%' THEN 4
		   WHEN pcode = '41%' THEN 5
		   WHEN pcode = '4%' THEN 6
		   ELSE NULL END =
				   (SELECT MIN(CASE WHEN pcode = '4130033' THEN 0
								    WHEN pcode = '413003%' THEN 1
								    WHEN pcode = '41300%' THEN 2
								    WHEN pcode = '4130%' THEN 3
								    WHEN pcode = '413%' THEN 4
								    WHEN pcode = '41%' THEN 5
								    WHEN pcode = '4%' THEN 6
								    ELSE NULL END)
					FROM PostalCode);
```

- 이 방법의 포인트는 7회 반복을 7회 CASE 식 분기로 변환했다는 것이다
- 이 쿼리는 아직 성능적인 관점에서 가장 좋은 답이라 하기에는 이르다
- 실행 계획을 보면 테이블 접근이 2회 발생하기 때문이다

### 윈도우 함수를 사용한 스캔 횟수 감소
- 위 쿼리는 순위의 최솟값을 서브쿼리에서 찾기 떄문에 테이블 스캔이 2회 발생한다
- 고전적이지만 다음과 같이 윈도우 함수를 사용하면 스캔 횟수를 줄 일 수 있다

```SQL
SELECT pcode,
	   district_name
FROM (SELECT pcode,
			 district_name,
			 CASE WHEN pcode = '4130033' THEN 0
				  WHEN pcode = '413003%' THEN 1
				  WHEN pcode = '41300%' THEN 2
				  WHEN pcode = '4130%' THEN 3
				  WHEN pcode = '413%' THEN 4
				  WHEN pcode = '41%' THEN 5
				  WHEN pcode = '4%' THEN 6
				  ELSE NULL END AS hit_code,
				  MIN(CASE WHEN pcode = '4130033' THEN 0
							WHEN pcode = '413003%' THEN 1
							WHEN pcode = '41300%' THEN 2
							WHEN pcode = '4130%' THEN 3
							WHEN pcode = '413%' THEN 4
							WHEN pcode = '41%' THEN 5
							WHEN pcode = '4%' THEN 6
							ELSE NULL END)
					OVER(ORDER BY CASE WHEN pcode = '4130033' THEN 0
										WHEN pcode = '413003%' THEN 1
										WHEN pcode = '41300%' THEN 2
										WHEN pcode = '4130%' THEN 3
										WHEN pcode = '413%' THEN 4
										WHEN pcode = '41%' THEN 5
										WHEN pcode = '4%' THEN 6
										ELSE NULL END) AS min_code
		FROM PostalCode) Foo
WHERE hit_code = min_code
;							
```

- 실행계획을 보면 테이블 접근이 1회로 감소한 것을 확인할 수 있다
- 그런데 윈도우 함수를 사용해 정렬이 추가로 사용되었다
- 여기서 비용이 발생하는데 테이블 크기가 크다면 테이블 풀 스캔을 줄이는 것이 효과가 더 크다

---

### 컬럼: 인덱스 온리 스캔
- SELECT 구문에서 사용하는 필드에 모두 인덱스가 포함되어 있을 때, 테이블 스캔을 하지 않고 인덱스를 사용한 접근만 실행할 수 있다
- Oracle 기준으로 실행계획에 'INDEX FAST FULL SCAN'이 일어나는 것을 확인할 수 있다
- 하지만 인덱스 온리 스캔은 SQL 구문에서 사용하는 필드가 적을 때만 사용할 수 있따
	- 따라서 사용할 수 있는 튜닝 상황이 많지 않다
	- 사용할 수 있는 상황이라도 필드가 많으면 효과가 그다지 좋지 않다

---

## 3.  반복 횟수가 정해지지 않은 경우
### 인접 리스트 모델과 재귀 쿼리
- 현재 주소와 과거 주소를 모두 관리하는 테이블을 생각해본다

**PostalHistory**

| name | pcode   | new_pcode |
| ---- | ------- | --------- |
| A    | 4130001 | 4130002   |
| A    | 4130002 | 4130003   |
| A    | 4130003 |           |
| ...  | ...     | ...       |

- 테이블에 저장된 데이터를 보고 A씨는 두 번 이사했다는 것을 알 수 있다
```Text
4130001 -> 4130002 -> 4130003(현재 주소)
```

- 이렇게 우편번호를 키로 삼아 데이터를 줄줄이 연결하는 것을 포인터 체인이라 한다
	- 포인터 체인은 데이터의 계층 구조를 표현하기 위한 고전적인 방법이다
- SQL에서 계층 구조를 찾는 방법 중 하나는 재귀 공통 테이블 식(recursion common table expression)이다

```SQL
WITH Explosion(name, pcode, new_pcode, depth) 
AS
(SELECT name, pcode, new_code, 1
 FROM PostalHistory
 WHERE name = 'A'
	 AND new_pcode IS NULL  -- 검색 시작
 UNION ALL
 SELECT child.name, child.pcode, child.new_pcode, depth + 1
 FROM Explosion parent, PostalHistory child
 WHERE parent.pcode = child.new_pcode
	 AND parent.name = child.name)

-- 메인 SELECT 구문
SELECT name, pcode, new_pcode
FROM Explosion
WHERE depth = (SELECT MAX(depth)
			   FROM Explosion)
```

**실행 결과**
| name | pcode  | new_pcode |
| ---- | ------ | --------- |
| A    | 413001 | 413002    |

- 실행 계획을 살펴보면 Explosion 뷰를 만들어 사용하고, 인덱스를 사용해 Nested Loops를 사용하기에 꽤 효율적인 계획이라고 볼 수 있다
- 그리고 이 방법은 표준 SQL에 포함되어 있는 내용이므로 구현에 의존적이지 않다
	- 다만 재귀 공통 테이블은 비교적 최근에 만들어진 기능으로 아직 구현되지 않았거나 실행 계획이 최적화 되지 않은 DBMS가 있을 수 있다

### 중첩 집합 모델
- SQL에서 계층 구조를 나타내는 방법
	1. 인접 리스트 모델
	2. 중첩 집합 모델
	3. 경로 열거 모델
- 1은 계층 구조를 설명하는 전통적인 방법이다
- 3은 갱신이 거의 발생하지 않은 경우에 힘을 발휘하는 방법이다
- 2는 각 레코드의 데이터를 집합으로 보고, 계층 구조를 집합의 중첩 관계로 나타낸다는 것이다

**PostalHistory2**
| name | pcode  | lft | rgt |
| ---- | ------ | --- | --- |
| A    | 413001 | 0   | 27  |
| A    | 413002 | 9   | 18  |
| A    | 413003 | 12  | 15  |
| ...  | ....   | ... | ... |

- lft와 rgt는 원의 왼쪽과 오른쪽 끝에 위치하는 좌표를 나타낸다
	- 좌표값은 대소 관계만 적절하다면 임의의 값을 사용할 수 있다
- 이사할 때마다 새로운 우편번호가 이전의 우편번호 '안에' 포함되는 형태로 추가한다
- 새로 삽입하는 우편번호의 좌표는 외측 원의 왼쪽 끝과 오른쪽 끝의 좌표를 사용해 결정하낟
- 외측의 우편번호 왼쪽 끝 좌표를 plft, 오른쪽 끝 좌표를 prgt라고 하면 다음과 같은 수식에 따라 자동적으로 노드의 좌표를 연산한다
	- 추가되는 노드의 왼쪽 끝 좌표 = (plft * 2 + prgt) / 3
	- 추가되는 노트의 오른쪽 끝 좌표 = (plft + prgt * 2) / 3
- A씨의 가장 오래된 주소를 간단한 SQL을 사용해 구할 수 있다
	- '왼쪽 끝의 좌표가 다른 모든 원의 왼쪽 끝 위치보다 작은' 이라는 조건을 사용한다
```SQL
SELECT name, pcode
FROM PostalHistory2 ph1
WHERE name = 'A'
	AND NOT EXISTS
		(SELECT *
		 FROM PostalHistory2 ph2
		 WHERE ph2.name = 'A'
		 AND ph1.lft > ph2.lft);
```

- Oracled에서 위 SQL의 실행계획을 보면, 외측 테이블(ph1)과 내측 테이블(ph2)을 한 번만 Nested Loops로 결합한다
	- 재귀 연산을 사용하지 않았다
- 중첩 집합의 코드가 재귀보다 따를지 단순하게 판단할 수는 없지만 일반적인 코딩에서는 없는 모델의 관점으로 문제를 해결할 수 있다는 것에 주목하자

