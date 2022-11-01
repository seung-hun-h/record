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


## 2. 순번을 이용한 테이블 분할
---
- 테이블을 여러 개의 그룹으로 분할하는 문제를 살펴본다

### 단절 구간 찾기
- 순번 테이블이 존재한다
- 순번 테이블에는 중간에 빠져있는 수들이 존재한다

**Numbers**
| num | 비고 |
| --- | ---- |
| 1   |      |
| 3   |      |
| 4   |      |
| 7   |      |

**기대 결과**
| gap_start | ~   | gap_end |
| --------- | --- | ------- |
| 2         | ~   | 2       |
| 5         | ~   | 6       |
| 10        | ~   | 11      |

### 집합 지향적 방법 - 집합의 경계선
- 전통적인 SQL에서는 집합지향적으로 생각해야 한다
```SQL
SELECT (N1.num + 1) AS gap_start,
       '~',
       (MIN(N2.num) - 1) AS gap_end
FROM Numbers N1 INNER JOIN Numbers N2
                    ON N2.num > N1.num
GROUP BY N1.num
HAVING (N1.num + 1) < MIN(N2.num);
```

- N2.num을 사용해 '특정 레코드의 값(N1.num)보다 큰 숫자의 집합'을 조건으로 지정했다
	- `ON N2.num > N1.num`
- N1.num + 1과 MIN(N2.num)의 값이 같지 않다는 것은 단절점이 존재한다는 의미다
- N1.num의 다음 숫자가 단절점의 시작(gap_start)이고 N2.num 바로 앞에 있는 숫자가 단절점의 끝(gap_end)이다

**실행계획**
![image](https://user-images.githubusercontent.com/60502370/199132996-b8279026-cb16-4153-a722-fea9ba6843bc.png)

- 실행계획을 보면 N1과 N2 사이에 결합이 발생하는 것을 알 수 있다
	- 책에는 Nested Loop 방식이 선택됐다
- 결합을 사용하는 쿼리는 비용이 높고 실행 계획 변동 위험을 안게된다

### 절차 지향적 방법 - '다음 레코드'와 비교
- '현재 레코드와 다음 레코드의 숫자 차이를 비교하고 차이가 1이 아니라면 사이에 비어있는 숫자가 있다' 라는 사고 방식은 절차 지향적 방식이다

```SQL
SELECT num + 1 AS gap_start,
       '~',
       (num + diff - 1) AS gap_end
FROM (SELECT num,
        MAX(num) OVER(ORDER BY num 
                        ROWS BETWEEN 1 FOLLOWING AND 1 FOLLOWING) - num AS diff
      FROM Numbers) TMP
WHERE diff <> 1;
```

- `MAX(num) OVER(ORDER BY num ROWS BETWEEN 1 FOLLOWING AND 1 FOLLOWING)`는 num의 다음 숫자를 반환한다
- diff는 테이블상  num의 다음 숫자와 num의 차이를 나타낸다

**실행계획**
![image](https://user-images.githubusercontent.com/60502370/199133381-62b2caa9-61a0-466c-bfdf-a0b6a924afbf.png)

- Numbers 테이블에 대한 접근이 한 번만 이루어지고, 결합이 사라져 성능이 안정적이다

## 3. 테이블에 존재하는 시퀀스 구하기
---
- 지금까지는 테이블에 존재하지 않는 시퀀스를 구했지만, 이번에는 테이블에 존재하는 수열을 그룹화하는 방법을 알아본다

### 집합 지향적 방법 - 다시, 집합의 경계선
- 집합 지향적으로 테이블에 존재하는 시퀀스를 구하는 것은 존재하지 않는 시퀀스를 구하는 것 보다 훨씬 간단하다
	- MAX/MIN 함수를 사용해서 시퀀스의 경계를 직접 구할 수 있기 때문이다

```SQL
SELECT MIN(num) AS low,
       MAX(num) AS high
FROM (SELECT N1.num AS num,
             COUNT(N2.num) - N1.num AS gp
      FROM Numbers N1 INNER JOIN Numbers N2
      ON N2.num <= N1.num
      GROUP BY N1.num) N
GROUP BY gp;
```
- 자기 결합으로 num 필드의 조합을 만들고 최대값과 최소값으로 집합의 경계를 구하는 방식이다
	- gp 값을 유일한 값이 될 거 같다(?)
	- 그래서 `GROUP BY gp`를 해도 버그가 생기지 않는다

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/199137762-470c8d27-4837-498e-ba0d-6024fb0cd4f8.png)

- N1과 N2에 자기 결합을 수행한다
- 극치 함수(MIN, MAX)로 집약을 수행한다
- 이때 정렬대신 Hash가 사용된다. 최근의 DBMS는 집약 함수 또는 극치 함수를 사용할 때 정렬이 아닌 해시를 사용한다

### 절차 지향적 방법 - 다시 '다음 레코드 하나'와 비교
- 기본적인 방식은 이전과 비슷하지만 코드가 길어진다

```SQL
SELECT low, high
FROM (SELECT low, 
             CASE WHEN high IS NULL
                  THEN MIN(high)
                        OVER(ORDER BY seq
                             ROWS BETWEEN CURRENT ROW
                                          AND UNBOUNDED FOLLOWING)
                  ELSE high END AS high
      FROM (SELECT CASE WHEN COALESCE(prev_diff, 0) <> 1
                        THEN num ELSE NULL END AS low,
                   CASE WHEN COALESCE(next_diff, 0) <> 1
                        THEN num ELSE NULL END AS high,
                   seq
            FROM (SELECT num,
                         MAX(num)
                         OVER(ORDER BY num
                              ROWS BETWEEN 1 FOLLOWING
                                   AND 1 FOLLOWING) - num AS next_diff,
                         num - MAX(num)
                                OVER(ORDER BY num
                                     ROWS BETWEEN 1 PRECEDING
                                          AND 1 PRECEDING) AS prev_diff,
                         ROW_NUMBER() OVER(ORDER BY num) AS seq
                  FROM Numbers) TMP1 ) TMP2 ) TMP3
WHERE low IS NOT NULL;
```

- 이 코드는 테이블에 존재하지 않는 시퀀스를 구할 때와 마찬가지로, 현재 레코드와 전후의 레코드를 비교한다

**TMP1**
```SQL
SELECT num,
     MAX(num)
     OVER(ORDER BY num
          ROWS BETWEEN 1 FOLLOWING
               AND 1 FOLLOWING) - num AS next_diff,
     num - MAX(num)
            OVER(ORDER BY num
                 ROWS BETWEEN 1 PRECEDING
                      AND 1 PRECEDING) AS prev_diff,
     ROW_NUMBER() OVER(ORDER BY num) AS seq
FROM Numbers
```

![image](https://user-images.githubusercontent.com/60502370/199138132-6713662b-18fa-4907-8ae8-cc144d99c326.png)
- `next_diff`는 다음 레코드의 num에서 현재 레코드의 num을 뺀 값이다
- `prev_diff`는 현재 레코드의 num에서 이전 레코드의 num을 뺀 값이다
- 따라서 `next_diff`가 1보다 크다면 현재 레코드와 다음 레코드 사이에 비어있는 부분이 존재한다는 것이다
	- `prev_diff`도 마찬가지다
- 이러한 성질을 이용하면 시퀀스의 단절 부분이 되는 양쪽 지점의 num을 구할 수 있다

**TMP2**
```SQL
SELECT CASE WHEN COALESCE(prev_diff, 0) <> 1
                        THEN num ELSE NULL END AS low,
                   CASE WHEN COALESCE(next_diff, 0) <> 1
                        THEN num ELSE NULL END AS high,
                   seq
FROM (SELECT num,
             MAX(num)
             OVER(ORDER BY num
                  ROWS BETWEEN 1 FOLLOWING
                       AND 1 FOLLOWING) - num AS next_diff,
             num - MAX(num)
                    OVER(ORDER BY num
                         ROWS BETWEEN 1 PRECEDING
                              AND 1 PRECEDING) AS prev_diff,
             ROW_NUMBER() OVER(ORDER BY num) AS seq
      FROM Numbers) TMP1 
```
![image](https://user-images.githubusercontent.com/60502370/199138526-185e8ba9-3822-433b-8d92-c70cbc0aba26.png)
- 3 ~ 4, 7 ~ 9처럼 동일한 레코드에 low필드와 high 필드가 존재하지 않는 시퀀스가 존재하므로 이를 정리해주고자 TMP3을 만들었다

**TMP3**
```SQL
SELECT low, high
FROM (SELECT low, 
             CASE WHEN high IS NULL
                  THEN MIN(high)
                        OVER(ORDER BY seq
                             ROWS BETWEEN CURRENT ROW
                                          AND UNBOUNDED FOLLOWING)
                  ELSE high END AS high
FROM (SELECT CASE WHEN COALESCE(prev_diff, 0) <> 1
                THEN num ELSE NULL END AS low,
           CASE WHEN COALESCE(next_diff, 0) <> 1
                THEN num ELSE NULL END AS high,
           seq
    FROM (SELECT num,
                 MAX(num)
                 OVER(ORDER BY num
                      ROWS BETWEEN 1 FOLLOWING
                           AND 1 FOLLOWING) - num AS next_diff,
                 num - MAX(num)
                        OVER(ORDER BY num
                             ROWS BETWEEN 1 PRECEDING
                                  AND 1 PRECEDING) AS prev_diff,
                 ROW_NUMBER() OVER(ORDER BY num) AS seq
          FROM Numbers) TMP1 ) TMP2 )
```
![image](https://user-images.githubusercontent.com/60502370/199138740-25027ed5-87dd-48b8-b18b-a17630ce222a.png)
- 마지막으로 가장 외측의 `WHERE low IS NOT NULL`로 불필요한 코드를 제거하여 최종 결과를 구할 수 있다

**실행계획**
![image](https://user-images.githubusercontent.com/60502370/199138917-5b5b9e0f-2c22-463c-a755-3739959a431c.png)
- 책에서는 서브쿼리에 대해서 정렬이 총 2번 발생하는데, 실습해보니 1번만 발생했다
- Oracle에서도 중간 결과를 메모리에 저장하므로 결과가 크면 역시 저장소를 사용한다
- 따라서 이 쿼리가 성능 측면에서 집합 지향 쿼리에 비해 좋은지는 서브쿼리에 의존하므로 단언할 수 없다
- 환경에 따라서 Numbers에 접근하는 수단으로 시퀀셜 스캔이 아닌 주 키의 인덱스를 사용한 인덱스 스캔이 발생할 수 있다. 실습이 그러하다