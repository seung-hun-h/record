# 레코드에 순번 붙이기
---
## 1. 기본 키가 한 개의 필드인 경우
- 학생의 체중을 저장하는 테이블이 있다
- 학생의 ID를 오름차순으로 순번 붙인다

**Weights**
![image](https://user-images.githubusercontent.com/60502370/197716548-73274585-d67b-425c-92e3-c501f22f7299.png)

### 윈도우 함수 사용
```SQL
SELECT student_id,
        ROW_NUMBER() OVER(ORDER BY student_id) AS seq
FROM Weights;
```
- 윈도우 함수를 사용하면 `Weights` 테이블에 한 번만 접근하고, 인덱스 스캔을 한다

### 상관 서브 쿼리 사용
```SQL
SELECT student_id,
        (SELECT COUNT(*)
         FROM Weights W2
         WHERE W2.student_id <= W1.student_id) AS seq
FROM Weights W1
;
```
- MySQL 같이 `ROW_NUMBER`를 사용할 수 없는 DBMS에서는 위 처럼 상관 서브쿼리를 사용한다
- 윈도우 함수를 사용하는 것과 기능은 동일하지만 테이블에 두 번 접근하기 때문에 윈도우 함수를 사용하는 경우 보다 성능은 떨어진다

## 2. 기본 키가 여러 개의 필드로 구성되는 경우
- 학급(class), 학생 ID(student_id)를 기본키로 하는 `Weights2` 테이블

**Weights**
![image](https://user-images.githubusercontent.com/60502370/197730416-bb4a8a2a-2376-43e4-95a3-dde7cf9088a3.png)

### 윈도우 함수 사용
```SQL
SELECT class, student_id,
        ROW_NUMBER() OVER(ORDER BY class, student_id) AS seq
FROM Weights2;
```
- 기본 키가 하나인 경우와 별반 다르지 않다

### 상관 서브쿼리 사용
```SQL
SELECT class, student_id,
        (SELECT COUNT(*)
         FROM Weights2 W2
         WHERE (W2.class, W2.student_id) <= (W1.class, W1.student_id)) AS seq
FROM Weights2 W1;
```
- 여러가지 방법이 있겠지만 가장 간단한 방법은 **다중 필드 비교**이다
	- 근데 오라클에서는 안되는듯

## 3. 그룹마다 순번을 붙이는 경우
- 이번에는 학급 마다 순번을 붙이는 경우다
**기대 결과**
![image](https://user-images.githubusercontent.com/60502370/197731425-384f5e5d-0c2d-44e5-8a8b-5926f25e4140.png)


### 윈도우 함수 사용
```SQL
SELECT class, student_id,
        ROW_NUMBER() OVER(PARTITION BY class ORDER BY student_id) AS seq
FROM Weights2;
```

### 상관 서브쿼리 사용
```SQL
SELECT class, student_id,
        (SELECT COUNT(*)
         FROM Weights2 W2
         WHERE W2.class = W1.class
            AND W2.student_id <= W1.student_id) AS seq
FROM Weights2 W1;
```

## 4. 순번과 갱신
- 갱신에서 순번을 매기는 방법을 알아본다
- `Weights2` 테이블을 변경해서 테이블에 순번(seq) 필드를 생성한다

### 윈도우 함수 사용
```SQL
UPDATE Weights2
SET seq = (SELECT seq
            FROM (SELECT class, student_id,
                        ROW_NUMBER() OVER(PARTITION BY class ORDER BY student_id) As seq
                  FROM Weights2) SeqTbl
            WHERE Weights2.class = SeqTbl.class
                AND Weights2.student_id = SeqTbl.student_id);

```

- `ROW_NUMBER`를 사용할 경우 상관 서브쿼리를 함께 사용해야 한다

### 상관 서브쿼리 사용
```SQL
UPDATE Weights2
SET seq = (SELECT COUNT(*)
		   FROM Weights2 W2
		   WHERE W2.class = Weights2.class
		   AND W2.student_id = Weights2.student_id);
```

