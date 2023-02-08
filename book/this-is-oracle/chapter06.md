# 6. PL/SQL 기본
## 6.1 SELECT 문
### 6.1.1 원하는 데이터를 가져와 주는 기본적인 `<SELECT ...  FROM>`
- SELECT는 데이터베이스 내의 테이블에서 원하는 정보를 추출하는 명령어이다
```SQL
[ WITH <Sub Query> ]
SELECT select_list
[ FROM table_source ] [ WHERE search_condition ]
[ GROUP BY group_by_expression ]
[ HAVING search_condition ]
[ ORDER BY order_expression [ ASC | DESC ] ]
```

### 6.1.2 특정한 조건의 데이터만 조회하는 `<SELECT ... FROM ... WHERE>`
**기본적인 WHERE 절**
```SQL
SELECT 필드 이름 FROM 테이블 이름 WHERE 조건식;
```

**관계 연산자 사용**
- 1970년 이후에 출생하고, 신장이 183 이상인 사람 조회

```SQL
SELECT userID, userName FROm user WHERE birthYear > 1970 AND height >= 183;
```

- 1970년 이후에 출생했거나, 신장이 182 이상인 사람 조회

```SQL
SELECT userID, userName FROm user WHERE birthYear > 1970 OR height >= 182;
```

**BETWEEN ... AND와 IN() 그리고 LIKE**
- 키가 180 ~ 183인 사람 조회
```SQL
SELECT userName, height FROM user WHERE height >= 180 AND height <= 183;

SELECT userName, height FROM user WHERE height BETWEEN 180 AND 183;
```

- 지역이 경남, 전남, 경북인 사람 조회

```SQL
SELECT userName, addr FROM user WHERE addr = '경남' OR addr = '전남' OR addr = '경북';

SELECT userName, addr FROM user WHERE addr IN ('경남''전남', '경북');
```
- 성이 김씨인 사람 조회
	- `%`는 그 뒤로 무엇이든 허용한다는 의미이다
```SQL
SELECT userName, height FROM user WHERE userName LIKE '김%';
```
- 이름이 '종신'인 사람 조회
	- `_`는 어떤 한 글자만 허용한다는 의미이다(이종신, 김종신, 한종신...)
```SQL
SELECT userName, height FROM user WHERE userName LIKE '_종신';
```

**ANY/ALL/SOM E 그리고 서브쿼리**
- '김경호'의 키보다 크거나 같은 사람

```SQL
-- 김경호의 키를 직접 입력하는 경우
SELECT userName, height FROM user WHERE height >= 177;

-- 서브 쿼리를 사용하는 경우 
SELECT userName, height FROM user 
	WHERE height >= (SELECT height FROM user WHERE userName = '김경호');
```

- 서브 쿼리가 둘 이상의 값을 반환하는 경우 에러가 발생할 수 있다

```SQL
SELECT userName, height FROM user 
	WHERE height >= (SELECT height FROM user WHERE addr = '경남');
```

- ANY는 서브 쿼리의 결과 중 하나라도 만족하면 된다
	- '= ANY(서브 쿼리)'의 결과는 'IN(서브 쿼리)'와 동일하다
```SQL
SELECT userName, height FROM user 
	WHERE height = ANY(SELECT height FROM user WHERE addr = '경남');
```

- ALL은 서브 쿼리의 여러 결과를 모두 만족해야 한다
```SQL
SELECT userName, height FROM user 
	WHERE height = ALL(SELECT height FROM user WHERE addr = '경남');
```

**원하는 순서대로 정렬하여 출력 :  ORDER BY**

- 'ORDER BY'를 사용해 결과를 정렬하여 출력한다
	- ASC는 오름차순, DESC는 내림차순이다
```SQL
SELECT userName, mDate FROM user ORDER BY mDate [ ASC | DESC];
```

**중복된 것은 하나만 남기는 DISTINCT**
- 사용자의 거주 지역을 파악할 경우 중복은 제거하는 것이 좋다
- 중복을 제거할 때 DISTINCT를 사용한다

```SQL
SELECT DISTINCT addr FROM user;
```

**ROWNUM 열과 SAMPLE 문**
- 입사일이 오래된 직원 5명의 employee_id를 조회하는 경우
```SQL
SELECT * 
FROM (SELECT employee_id, hire_date
        FROM employees
        ORDER BY hire_date ASC)
WHERE ROWNUM <= 5; -- 5건만 조회
```
- `ROWNUM`열은 SELECT 문을 조회하면 자동으로 그 앞에 순차로 붙는 임시 열이다
- SELECT를 실행하는 시점에 생성이 되고 순차번호가 부여된다
- 위 쿼리는 정상적으로 결과를 반환하지만 Oracle 성능에 매우 나쁜 영향을 미칠 수 있다
	- 서브쿼리에서 모든 결과를 조회하고 정렬한 후 5개만 반환하기 때문이다

- 만약 입사일에 관계없이 앞에서 5명을 조회하는 경우 다음과 같이 사용하면 된다
	- 항상 테이블에 저장된 상위 5건만 조회한다
```SQL
SELECT employee_id, hire_date
FROM employees
WHERE ROWNUM <= 5;
```

- 임의의 데이터를 추출하고 싶은 경우 `SAMPLE`을 사용한다
```SQL
SELECT employee_id, hire_date
FROM employees SAMPLE(5);
```

**테이블을 복사하는 `CREATE TABLE ... AS SELECT`**
- `CREATE TABLE ... AS SELECT`는 테이블을 복사해서 사용하는 경우 주로 사용된다
	- PK나 FK등 제약 조건은 복사되지 않는다
```SQL
CREATE TABLE 새로운 테이블 AS (SELECT 복사할 열 FROM 기존 테이블)
```

### 6.1.3 GROUP BY 및 HAVING 그리고 집계 함수
**GROUP BY절**
- 각 사용자별로 구매한 개수를 합쳐서 출력
```SQL
SELECT userID, SUM(amount)
FROM buy
GROUP BY userID;
```

**집계 함수**
- AVG()
- MIN()
- MAX()
- COUNT()
- COUNT(DISTINCT)
	- 행의 개수를 센다. 중복은 1개만 인정
- STDEV()
- VARIANCE()

- 휴대폰이 있는 사용자의 수를 카운트

```SQL
SELECT * FROM user -- 전체 사용자 수 카운트
SELECT COUNT(mobile1) FROM user; -- mobile1이 NULL인 사용자는 카운트에 제외
```

**HAVING 절**
- 총 구매액이 1,000 이상인 사용자만 조회
```SQL
SELECT userID, SUM(price * amount)
FROM buy
WHERE SUM(price * amount) > 1000
GROUP BY userID;
```

- 위 코드는 예외가 발생한다
	- WHERE 절에는 집계 함수를 사용할 수 없다
- HAVING은 집계 함수에 대한 조건을 제한한다
	- HAVING은 항상 GROUP BY 다음에 온다

```SQL
SELECT userID, SUM(price * amount)
FROM buy
GROUP BY userID;
HAVING SUM(price * amount) > 1000; -- HAVING은 항상 GROUP BY 다음에 온다
```

**ROLLUP(), GROUPING_ID(), CUBE() 함수**
- 총합 또는 중간 합계가 필요하다면 GROUP BY절과 함께 ROLLUP() 또는 CUBE()를 사용하면 된다

```SQL
SELECT idNum, groupName, SUM(price * amount)
FROM buy
GROUP BY ROLLUP(groupName, idNum);
```

```SQL
SELECT prodName, color, SUM(amount)
FROM cubeTb1
GROUP BY CUBE(color, prodName)
ORDER BY prodName, color;
```

### 6.1.4 WITH 절과 CTE
- WITH절은 CTE(Common Table Expression)을 표현하기 위한 구문이다
- CTE는 기존의 뷰, 파생 테이블, 임시 테이블 등으로 사용되던 것을 대신할 수 있으며, 더 간결한 식으로 보여지는 장점이 있다
- CTE는 비재귀적 CTE와 재귀적 CTE로 나뉜다

**비재귀적 CTE**
- 말 그대로 재귀적이지 않은 CTE이다
- 단순한 형태이며 복잡한 쿼리 문장을 단순화 시키는데 적합하다

```SQL
WITH CTE_테이블이름(열 이름)
AS
(
	쿼리문
)
SELECT 열 이름 FROM CTE_테이블이름;
```

- buyTbl의 총 구매액을 구하는 쿼리

```SQL
SELECT userID, SUM(price * amount)
FROM buy
GROUP BY userID;
```
- 총 구매액에 많은 순서대로 정렬하고 싶은 경우 앞의 쿼리에 이어서 ORDER BY를 작성해도 되지만, SQL이 더욱 복잡해 보일 수 있다
- 위 쿼리의 결과가 abc 테이블이라고 가정하면 쿼리는 훨씬 단순해질 수 있다

```SQL
SELECT *
FROM abc
ORDER BY "총 구매액" DESC;
```

- 이를 WITH를 사용한 쿼리로 작성하면 아래와 같다
```SQL
WITH abc(userID, total)
AS
(
	SELECT userID, SUM(price * amount)
	FROM buy
	GROUP BY userID
) 
SELECT * 
FROM abc
ORDER BY total DESC;
```

- CTE는 뷰와 용도가 비슷하지만 개선된 점이 많다
- 뷰는 계속해서 존재해 다른 구문에서 사용할 수 있지만, CTE는 구문이 끝나면 같이 소멸된다
- CTE는 다음과 같이 중복 CTE가 허용된다
	- CCC의 쿼리 문에서는 AAA, BBB를 참조할 수 있지만, BBB 쿼리문에서는 CCC를 참조할 수 없다

```SQL
WITH
AAA(userID, total)
	AS (SELECT userID, SUM(price * amount) FROM buy GROUP BY userID)),
BBB(sumtotal)
	AS (SELECT SUM(total) FROM AAA),
CCC(sumavg)
	AS (SELECT sumtotal / (SELECT count(*) FROM buy FROM BBB)),
SELECT * FROM CCC;
```

**재귀적 CTE**
- 재귀적이라는 의미는 자기자신을 반복적으로 호출한다는 것이다

```SQL
WITH CTE_테이블이름(열이름)
AS
(
	<쿼리문1 : SELECT * FROM 테이블A>
	UNION ALL
	<쿼리문2 : SELECT * FROM 테이블 A JOIN CTE_테이블이름>
)
SELECT * FROM CTE_테이블이름;
```
1. <쿼리문1>을 실행한다. 이것이 전체 흐름의 최초 호출에 해당한다. 그리고 레벨의 시작은 0으로 초기화 된다
2. <쿼리문2>를 실행한다. 레벨을 레벨 + 1로 증가시킨다. SELECT의 결과가 빈 것이 아니라면 'CTE_테이블이름'을 다시 재귀적으로  호출한다
3. 계속 2번을 반복한다. 단 SELECT의 결과가 아무것도 없다면 재귀적인 호출이 중단된다
4. 외부의 SELECT 문을 실행해서 앞 단계의 누적된 결과를 가져온다

```SQL
WITH empCTE(empName, mgrName, dept, empLevel)
AS
(
	(SELECT emp, manager, department, 0
		FROM empTbl
		WHERE manager = '없음') -- 상관이 없는 사람이 사장
	UNION ALL
	(SELECT empTbl.emp, empTbl.manager, empTbl,department, embCTE.empLevel + 1
		FROM empTbl INNER JOIN empCTE
			ON empTbl.manager = empCTE.empName)
)
SELECT * FROM empCTE ORDER BY dept, empLevel;
```

### 6.5.1 SQL의 분류
**DML**
- 데이터 조작어
- 선택, 삽입, 수정, 삭제 하는데 사용되는 언어이다
- SELECT, INSERT, DELETE, UPDATE
**DDL**
- 데이터 정의어
- 데이터베이스, 테이블, 뷰, 인덱스 등 데이터베이스 개체를 생성/삭제/변경 하는 언어이다
- CREATE, DROP, ALTER
**DCL**
- 데이터 제어어
- 사용자에게 어떤 권한을 부여하거나 빼앗을 때 사용하는 언어이다
- GRANT, REVOKE, DENY

## 6.2 데이터 변경을 위한 SQL 문
### 6.2.1 데이터 삽입: INSERT
**INSERT문 기본**
- 생략

**자동으로 증가하는 시퀀스**
- 시퀀스 생성
```SQL
CREATE SEQUENCE idSEQ
START WITH 1 -- 시작값
INCREMENT BY 1 -- 증가값
```
- 시퀀스 사용
	- '시퀀스이름.NEXTVAL'을 사용하면된다
```SQL
INSERT INTO testTbl VALUES (idSEQ.NEXTVAL, '유나', 25, DEFAULT)
```
- 시퀀스의 현재 값 확인
	- DUAL 테이블은 Oracle에 내장된 가상의 테이블이다
	- 용도는 SELECT가 특별히 테이블을 조회할 필요가 없더라도 구문 형식상 FROM이 반드시 있어야 하므로 FROM절 뒤에 그냥 써준다고 생각하면 된다
```SQL
SELECT idSEQ.CURRVAL FROM DUAL;
```
- 특정 범위의 값이 계속 반복되어서 입력되도록 할 수도 있다
```SQL
CREATE SEQUENCE cycleSEQ
	START WITH 100
	INCREMENT BY 100
	MINVALUE 100
	MAXVALUE 300
	CYCLE -- 반복 설정
	NOCACHE -- 캐시 사용 안함
;
```

**대량의 샘플 데이터 생성**
- INSERT INTO ... SELECT 구문을 사용하면 다른 테이블의 데이터를 가져와서 대량으로 입력하는 효과를 낸다

```SQL
CREATE TABLE testTbl (
empID NUMBER(6),
FirstName VARCHAR2(20),
LastName VARCHAR2(25),
Phone VARCHAR2(20));

INSERT INTO testTbl
	SELECT EMPLOYEE_ID, FIRST_NAME, LAST_NAME, PHONE_NUMBER
	FROM HR.employees;
```

- 테이블의 정의까지 생략하고 싶으면 아래와 같이 작성할 수 있다

```SQL
CREATE TABLE testTbl AS
	(SELECT EMPLOYEE_ID, FIRST_NAME, LAST_NAME, PHONE_NUMBER
		FROM HR.employees);
```

### 6.2.2 데이터의 수정: UPDATE
- 생략

### 6.2.3 데이터의 삭제: DELETE FROM
- DELETE는 행 단위로 삭제한다
- DML 문인 DELETE는 트랜잭션 로그를 기록하는 작업 때문에 삭제하는데 오래 걸린다
- DDL 문인 DROP문은 테이블 자체를 삭제한다. DDL은 트랜잭션을 발생하지 않는다
- DDL 문인 TRUNCATE문의 효과는 DELETE문과 동일하지만 트랜잭션 로그를 기록하지 않아서 속도가 빠르다
- 대용량의 테이블의 전체 내용을 삭제할 때 테이블이 필요하지 않는 경우 DROP을 사요하고, 테이블 구조는 남겨 놓고 싶다면 TRUNCATE를 사용한다

### 6.2.4 조건부 데이터 변경: MERGE
- MERGE는 하나의 문장에서 경우에 따라 INSERT, UPDATE, DELETE를 수행할 수 있다

```SQL
MERGE INTO member M -- 변경될 테이블 (target 테이블)
-- 변경할 기준이 되는 테이블(source 테이블)
	USING (SELECT changeType, userID, userName, addr FROM changeTbl) C
	ON (M.userID, C.userID) -- userID를 기준으로 두 테이블을 비교한다
WHEN MATCHED THEN
	UPDATE SET M.addr = C.addr
	DELETE WHERE C.changeType = '회원탈퇴'
WHEN NOT MATCHED THEN
	INSERT (userID, userName, addr) VALUES (C.userID, C.userName, C.addr);
```
