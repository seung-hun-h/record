# 레코드에 순번 붙이기 응용
---
- 레코드에 순번을 붙을 수 있으면, SQL 에서 자연 수열의 성질을 이용해서 다양한 테크닉을 활용할 수 있다

## 1. 중앙값 구하기
- 중앙값이란 숫자를 정렬하고 양쪽 끝에서부터 수를 세는 경우 정중앙에 위치한 값을 말한다
- 테이블에 저장된 레코드의 수가 홀수인 경우 중앙값은 하나이고, 짝수인 경우 중앙값은 두개가 된다
- 23강에서 다루었던 `Weights` 테이블을 사용한다

### 집합 지향적인 방법
- 테이블을 상위 집합과 하위 집합으로 나누고 공통 부분을 검색하는 방법이다

```SQL
SELECT AVG(weight)
FROM (SELECT W1.weight
      FROM Weights W1, Weights W2
      GROUP BY W1.weight
      HAVING SUM(CASE WHEN W2.weight >= W1.weight THEN 1 ELSE 0 END) >= COUNT(*) / 2
         AND SUM(CASE WHEN W2.weight <= W1.weight THEN 1 ELSE 0 END) >= COUNT(*) / 2) TMP;


```
- 자기 결합을 사용해서 상위 집합과 하위 집합으로 나누었다
- AVG 함수는 레코드의 수가 짝수인 경우를 고려하여 사용했다

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/197908938-37828c27-5d87-48b9-a447-a9282c565268.png)

- `Weights` 테이블에 대한 접근이 2회 발생했다
- 그리고 크로스 조인이 발생헀다
- 결합은 비용이 크고 불안정하다는 단점이 있다

### 절차 지향적 방법1 - 세계의 중심을 향해
- SQL에서 자연수의 특징을 활용하면 '양쪽 끝부터 숫자 세기'를 할 수 있다
- 양쪽 끝에서 사람이 서로를 마주 보며 걸어오고, 결국 두 사람이 만나는 지점이 '세계의 중심'이 되는 것이다

```SQL
SELECT AVG(weight) AS median
FROM (SELECT weight,
           ROW_NUMBER() OVER(ORDER BY weight ASC, student_id ASC) AS hi,
           ROW_NUMBER() OVER(ORDER BY weight DESC, student_id DESC) AS lo
      FROM Weights) TMP
WHERE hi IN (lo - 1, lo, lo + 1);  
```
- 홀수의 경우에는 hi = lo가 되어 중심점이 반드시 하나만 존재한다
- 짝수의 경우에는 hi = lo - 1 혹은 hi = lo + 1 두개가 존재한다
- 홀수와 짝수인 경우의 조건 분기를 IN 연산자로 한꺼번에 수행한다

**주의점**
- 순번을 붙일때는 항상 `ROW_NUMBER()`를 사용해야 한다. 레코드 집합에 자연수의 집합을 할당해서 연속성과 유일성을 갖게 만들 수 있다
- `ORDER BY`키에 `weight`뿐 아니라 `student_id`도 포함해야 한다
	- 아마도 중복되는 값에 대해서 ASC가 DESC의 역순이 되는 것을 보장할 수 없기 때문인듯(?)

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/197909994-8967cf21-0c85-4431-9929-7bf2bfd187fd.png)
- 테이블 접근이 1회로 줄었지만 정렬이 2회 수행된다
- 이러한 실행 계획은 테이블의 크기가 클 수록 트레이드 오프의 이득이 커진다

### 절차 지향적 방법 - 2 빼기 1은 1

```SQL
SELECT AVG(weight)
FROM (SELECT weight,
            2 * ROW_NUMBER() OVER(ORDER BY weight) - COUNT(*) OVER() AS diff
      FROM Weights) TMP
WHERE diff BETWEEN 0 AND 2;
```
- 순번은 연속적이므로 레코드의 수가 홀수인 경우 `중앙값 - 레코드수/2 = 0.5`를 항상 만족한다. 따라서 2를 곱하면 1이된다
- 레코드의 수가 짝수인 경우 중앙값이 두 개이다. 두 개의 중앙값과 레코드수/2를 뺀 결과는 0 혹은 1이다. 따라서 2를 곱하면 0과 2가된다
- 결과적으로 레코드의 수가 홀수, 짝수인 경우를 모두 포함하기 위해서는 diff의 값을 0-2가 되어야 한다

**실행 결과**
![image](https://user-images.githubusercontent.com/60502370/197910772-3265a559-d987-40ff-9555-62ab17fc989f.png)
- 테이블 접근 1회, 정렬 1회의 실행 계획이 나타난다
- SQL 표준으로 중앙값을 구하는 가장 빠른 쿼리이다

