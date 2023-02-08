# 13강 자르기
---
- GROUP BY는 '집약' 외에도 '자르기' 기능이 있다
	- '자르기'는 원래 모집합인 테이블을 작은 부분 집합들로 분리하는 것이다
- GROUP BY가 집약, 자르기 두 가지 기능을 가지기 때문에 이해하기 어려울 수 있다

## 1. 자르기와 파티션
- 다음과 같은 EMPLOYEES 테이블이 있다고 가정한다

**EMPLOYEES**
<img width="800" alt="Screen Shot 2022-10-02 at 9 45 21 AM" src="https://user-images.githubusercontent.com/60502370/193433280-4b725af1-bcef-4905-9c3d-e0e606e3d34e.png">
- first_name의 첫 글자를 사용해 특정한 알파벳으로 시작하는 이름을 가진 사람이 몇명인지 집계해본다
- 집합의 요소 수를 구하기 위해 COUNT를 사용하고, first_name의 앞 글자를 GROUP BY 구의 키로 지정한다

```SQL
SELECT SUBSTR(first_name, 1, 1) AS label, COUNT(employee_id)
FROM employees
GROUP BY SUBSTR(first_name, 1, 1)
;
```

**쿼리 실행 결과**
<img width="278" alt="image" src="https://user-images.githubusercontent.com/60502370/193433393-69e5cc23-f7b5-427d-9bc5-26a8a6a13d0c.png">

### 파티션
- GROUP BY 구로 잘라 만든 하나하나의 부분 집합을 수학적으로는 '파티션(partition)'이라 한다
- 파티션은 서로 중복되는 요소를 가지지 않는 부분집합이다
- EMPLOYEES 테이블에서 salary가 4000 미만이면 'Low', 4000이상 8400미만이면 'Middle', 8400 이상이면 'High'로 나누어 보자
- GROUP BY의 키를 세 부분으로 나누기 위해 CASE 식을 사용한다

```SQL
SELECT CASE WHEN salary < 4000 THEN 'Low'
              WHEN salary BETWEEN 4000 AND 8400 THEN 'Middle'
              WHEN salary > 8400 THEN 'High'
              ELSE NULL END AS salary_class,
       COUNT(*)
FROM employees
GROUP BY CASE WHEN salary < 4000 THEN 'Low'
              WHEN salary BETWEEN 4000 AND 8400 THEN 'Middle'
              WHEN salary > 8400 THEN 'High'
              ELSE NULL END
;
```

- PostgreSQL, MySQL은 GROUP BY 구의 CASE 식을 'salary_class'로 대체할 수도 있다
	- 하지만 이는 표준에는 없는 내용이다
	- 오라클은 안되는 듯 하다

**실행 계획**
<img width="800" alt="image" src="https://user-images.githubusercontent.com/60502370/193433593-19aed8d4-f6bf-4973-a58d-9cb39e346011.png">
- 실행계획을 보면 알 수 있듯이 GROUP BY 구에서 CASE 식 또는 함수를 사용해도 실행 계획에는 영향이 없다
	- 단순한 필드가 아닌 필드에 연산을 추가하면 어느정도 CPU 오버 헤드가 걸릴 것이다
- 집약 함수와 GROUP BY의 실행 계획은 성능적인 측면에서, 해시(또는 정렬)에 사용되는 워킹 메모리의 용량에 주의하라는 것 이외에 따로 할 말은 없다

### BMI로 자르기
- 책의 예제 중 BMI 수치에 따라 자르는 예제가 있다
```SQL
SELECT CASE WHEN weight / POWER(height/100, 2) < 18.5 THEN '저체중'
			WHEN 18.5 <= weight / POWER(height/100, 2)
				AND weight / POWER(height/100, 2) < 25 THEN '정상'
			WHEN 25 <= weight / POWER(height/100, 2) THEN '과체중'
			ELSE NULL END AS bmi,
		COUNT(*)
FROM Persons
GROUP BY CASE WHEN weight / POWER(height/100, 2) < 18.5 THEN '저체중'
			WHEN 18.5 <= weight / POWER(height/100, 2)
				AND weight / POWER(height/100, 2) < 25 THEN '정상'
			WHEN 25 <= weight / POWER(height/100, 2) THEN '과체중'
			ELSE NULL END
;
```

- GROUP BY에 필드뿐 아니라 **복잡한 수식을 기준으로 자를 수 있다**는 것을 기억하자

## 2. PARTITION BY 구를 사용한 자르기
- PARTITION BY는 GROUP BY 구에서 집약 기능을 제외하고 자르기 기능만 남긴 윈도우 함수이다
- PARTITION BY를 사용해도 필드뿐 아니라 CASE 식, 계산 식을 사용한 복잡한 기준을 사용할 수 있다
- EMPLOYEES 테이블에서 salary를 Low, Middle, High로 나누고 각 등급 내부에서 순위를 매기는 코드는 다음처럼 작성할 수 있다

```SQL
SELECT employee_id, 
       first_name,
       salary,
       CASE WHEN salary < 4000 THEN 'Low'
              WHEN salary BETWEEN 4000 AND 8400 THEN 'Middle'
              WHEN salary > 8400 THEN 'High'
              ELSE NULL END AS salary_class,
       RANK() OVER(PARTITION BY CASE WHEN salary < 4000 THEN 'Low'
                                     WHEN salary BETWEEN 4000 AND 8400 THEN 'Middle'
                                     WHEN salary > 8400 THEN 'High' ELSE NULL END
                   ORDER BY salary DESC) AS salary_rank_in_class
FROM employees
ORDER BY salary_class, salary_rank_in_class
;
```

**실행 결과**
<img width="646" alt="image" src="https://user-images.githubusercontent.com/60502370/193433850-c4dff82d-e0ed-4c45-84fa-ea806f72d2ef.png">
- PARTITION BY는 GROUP BY와 달리 집약 기능이 없어 레코드가 모두 원래 형태 그대로 나오는 것을 알 수 있다

---
## 번외: Oracle 쿼리 실행 순서
![image](https://user-images.githubusercontent.com/60502370/193433878-f1865b5b-61a9-407c-91fe-a13ef1a70f26.png)
- FROM > CONNECT BY > GROUP BY > HAVING > WHERE > SELECT > ORDER BY
- GROUP BY에서는 Alias를 사용할 수 없는 이유가 SELECT 이전에 실행되기 때문인 것 같다
- ORDER BY에서 Alias를 사용할 수 있는 이유가 SELECT 이후에 실행되기 때문인 것 같다

- 참고: https://oracle.readthedocs.io/en/latest/sql/basics/query-processing-order.html