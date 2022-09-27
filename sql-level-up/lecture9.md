# 9강 집계와 조건 분기
---
- 집계를 수행하는 쿼리를 작성할 때, 쓸데 없이 길어지는 경우를 자주 볼 수 있다
- 다음은 지역별 인구를 기록하는 Population 테이블이다
	- 성별 1은 남성, 2는 여성을 나타낸다

| prefecture | sex | pop |
| ---------- | --- | --- |
| 성남       | 1   | 60  |
| 성남       | 2   | 40  |
| ...        | ... | ... |

- 지역별 남성과 여성 인구를 구하는 것을 목표로 한다

| prefecture | pop_men | pop_wom |
| ---------- | ------- | ------- |
| 수원       | 90      | 100     |
| 일산       | 100     | 100     |
| 성남       | 60      | 40      |
| ...        | ...     | ...     |

## 1. 집계 대상으로 조건 분기

### UNION을 사용한 방법
- 절차적 사고 방식으로 일단 남성의 지역별 인구를 구하고 그 다음 여성의 지역별 인구를 구한 후 머지하는 방식을 떠올릴 수 있다

```SQL
SELECT prefecture, SUM(pop_mem) AS pop_men, SUM(pop_wom) AS pop_wom
FROM (
	SELECT prefecture, pop AS pop_men, NULL AS pop_wom
	FROM Population
	WHERE sex = '1'
	UNION
	SELECT prefecture, NULL AS pop_men, pop AS pop_wom
	FROM Population
	WHERE sex = '2') TMP
GROUP BY prefecture;
```

- 위 쿼리를 실행하면 원하는 결과를 얻을 수 있다

### UNION의 실행 계획
- 하지만 실행 계획을 살펴보면 테이블 풀 스캔을 2회 수행 한다

### 집계의 조건 분기도 CASE 식을 이용
- 이 문제는 CASE 식의 응용 방법으로 굉장히 유명한 표측/표두 레이아웃 이동 문제이다
- CASE 식을 집약 함수 내부에 포함시켜서 남성 인구와 여성 인구 필터를 만든다

```SQL
SELECT prefecture,
	SUM(CASE WHEN sex = '1' THEN pop ELSE 0) as pop_men,
	SUM(CASE WHEN sex = '2' THEN pop ELSE 0) as pop_wom
FROM Population
GROUP BY prefecture;
```

- 위 처럼 쿼리가 굉장히 간단해진다

### CASE 식의 실행 계획
- CASE 식을 집계 함수 내부에 사용하면 실행 계획도 간단하게 된다
- 테이블 풀 스캔이 1회로 감소 한다
---
- CASE 식으로 조건 분기를 잘 이용하면 UNION을 사용하지 않을 수 있다

## 2. 집약 결과로 조건 분기
- 집약에 조건 분기를 적용하는 또 하나의 패턴으로, 집야 결과에 조건 분기를 수행하는 경우가 있다
- 다음은 직원과 직원이 소속된 팀을 관리하는 테이블 Employees이다

| emp_id | team_id | emp_name | team      |
| ------ | ------- | -------- | --------- |
| 201    | 1       | Joe      | 상품 기획 |
| 201    | 2       | Joe      | 개발      |
| ...    | ...     | ...      | ...       |

 - 다음과 같은 조건에 맞춰 결과를 만드는 것을 목표로 한다
	 - 소속된 팀이 1개라면 해당 직원은 팀의 이름을 그대로 출력한다
	 - 소속된 팀이 2개라면 해당 직원은 '2개팀 겸무'라는 문자열을 출력한다
	 - 소속된 팀이 3개 이상이라면 해당 직원은 '3개 이상을 겸무'라는 문자열을 출력한다

| emp_name | team            |
| -------- | --------------- |
| Jim      | 개발            |
| Bree     | 3개 이상을 겸무 |
| Joe      | 3개 이상을 겸무 |

### UNION을 사용한 조건 분기

```SQL
SELECT emp_name,
	MAX(team) AS team -- Why MAX? -> GROUP BY 때문에 team은 에러남
FROM Employees
GROUP BY emp_name
HAVING COUNT(*) = 1
UNION
SELECT emp_name,
	'2개를 겸무' AS team
FROM Employees
GROUP BY emp_name
HAVING COUNT(*) = 2
UNION
SELECT emp_name,
	'3개 이상을 겸무' AS team
FROM Employees
GROUP BY emp_name
HAVING COUNT(*) >= 3
;
```

- 이 쿼리는 조건 분기가 레코드 값이 아닌, 집합의 레코드 수에 적용된다
	- WHERE 구가 아닌 HAVING 구에 조건 분기가 지정되어있다
- 하지만 UNION으로 머지하고 있는 이상 별반 다르지 않다


### UNION의 실행 계획
- 3 번의 테이블 풀 스캔을 한다

### CASE 식을 사용한 조건 분기

```SQL
SELECT emp_name,
		CASE WHEN COUNT(*) = 1 THEN MAX(team)
			 WHEN COUNT(*) = 2 THEN '2개를 겸무'
			 WHEN COUNT(*) >= 3 THEN '3개 이상을 겸무'
		END AS team
FROM Employees
GROUP BY emp_name;
```

### CASE 식을 사용한 조건 분기의 실행 계획
- CASE 식을 사용하면 테이블에 접근 비용을 3분의 1로 줄일 수 있다
- 추가적으로 GROUP BY의 HASH 연산도 3회에서 1회로 줄었다
- COUNT 또는 SUM과 같은 집약 함수의 결과는 1개의 레코드로 압축된다
	- 집약 함수의 결과가 스칼라(더 이상 분할 불가능한 값)이 되는 것이다
- 따라서 CASE 식의 매개변수에 집약 함수를 넣을 수 있다

---
- WHERE 구뿐만 아니라 HAVING 구에 조건 분기를 하는 사람도 초보자이다