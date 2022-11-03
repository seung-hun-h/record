# 레코드에서 필드로의 갱신
---
- 학생의 시험 점수를 레코드로 가진 테이블과 필드로 가진 두 개의 테이블이 있다

**점수를 레코드로 갖는 테이블(ScoreRows)**
![image](https://user-images.githubusercontent.com/60502370/199647276-e867dd95-5cfd-419f-a4e9-5c4a254b8fa7.png)

**점수를 필드로 갖는 테이블(ScoreCols)**
![image](https://user-images.githubusercontent.com/60502370/199647325-1c7726d9-574f-4535-b058-d54e4e13a8a3.png)

- ScoreCols의 점수를 채워넣어 본다

## 1. 필드를 하나씩 갱신
- 가장 기본적으로 한 과목씩 갱신하는 SQL을 생각할 수 있다
- 간단하고 명료하지만 3개의 상관 서브쿼리를 실행해야 한다는 단점을 가지고 있다

```SQL
UPDATE ScoreCols
SET score_en = (
		SELECT score
		FROM ScoreRows sr
		WHERE sr.student_id = sr.student_id
		AND subject = '영어'
	),
	score_nl = (
		SELECT score
		FROM ScoreRows sr
		WHERE sr.student_id = sr.student_id
		AND subject = '국어'
	),
	score_mt = (
		SELECT score
		FROM ScoreRows sr
		WHERE sr.student_id = sr.student_id
		AND subject = '수학'
	)
;
```

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/199647954-e2ff6596-b359-4ef2-96bd-e4965fa4abde.png)

- ScoreRows 테이블에 대한 접근이 3번 이루어진다
	- 이는 과목이 늘어날 수록 접근 횟수도 같이 늘어난다
	- 상관 서브쿼리를 또 작성해야 하기 때문이다

## 2. 다중 필드 할당
- 다중 필드 할당을 사용하면 상관 서브쿼리가 가진 단점을 해결할 수 있다

```SQL
UPDATE ScoreCols
SET (score_en, score_nl, score_mt)
	= (SELECT MAX(CASE WHEN subject = '영어'
				  	   THEN score
				  	   ELSE NULL END) AS score_en,
		  	  MAX(CASE WHEN subject = '국어'
				  	   THEN score
				  	   ELSE NULL END) AS score_nl,
		  	  MAX(CASE WHEN subject = '수학'
				  	   THEN score
				  	   ELSE NULL END) AS score_mt
  	   FROM ScoreRows SR
  	   WHERE SR.student_id = ScoreCols.student_id);
```

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/199648199-3a5f9137-1c24-4043-849d-8fbe9ca41d6d.png)
- 이렇게 하면 서브쿼리를 한 번에 처리할 수 있어 성능도 향상되고 코드도 간단해진다
- 갱신해야할 필드의 수가 늘어나도, 서브쿼리의 수가 늘어나지 않아 성능적으로 염려되는 부분이 없다

#### 다중 필드 할당과 스칼라 서브 쿼리
- 다중 필드 할당
	- SET 구의 왼쪽을 보면 영어, 국어, 수학 세 개의 필드를 `(score_en, score_nl, score_mt)`와 같은 리스트 형식으로 지정했따
	- 이렇게 하면 전체를 하나의 조작 단위로 만들 수 있다
- 스칼라 서브쿼리
	- 서브쿼리 내부를 보면 CASE 식으로 과목별 점수를 검색한다. 이떄 중요한 것은 각각의 점수에 MAX 함수를 적용하는 것이다
	- 학생 A001의 score_en 필드 경우 MAX 함수를 적용하지 않으면 (100, NULL, NULL)처럼 3개의 값이 리턴되므로 MAX 함수를 적용해야 한다


## 3. NOT NULL 제약이 걸려있는 경우
- ScoreColsNN 테이블은 각 과목 점수 필드에 NOT NULL 제약 조건이 걸려있다

**ScoreColsNN**
![image](https://user-images.githubusercontent.com/60502370/199648645-ad26a0a6-3af0-4f70-aca0-497f3e059488.png)

### UPDATE 구문
```SQL
UPDATE ScoreColsNN
SET score_en = COALESCE (( -- 학생은 있지만 과목이 없을 때 NULL 대응
		SELECT score
		FROM ScoreRows sr
		WHERE sr.student_id = sr.student_id
		AND subject = '영어'
	), 0),
	score_nl = COALESCE ((
		SELECT score
		FROM ScoreRows sr
		WHERE sr.student_id = sr.student_id
		AND subject = '국어'
	), 0),
	score_mt = COALESCE ((
		SELECT score
		FROM ScoreRows sr
		WHERE sr.student_id = sr.student_id
		AND subject = '수학'
	), 0)
WHERE EXISTS (SELECT *  -- 처음부터 학생이 존재하지 않은 때의 NULL 대응
			  FROM ScoreRows
			  WHERE student_id = ScoreColsNN.student_id)
;
```

```SQL
UPDATE ScoreColsNN
SET (score_en, score_nl, score_mt)
	= (SELECT COALESCE (MAX(CASE WHEN subject = '영어'
				  	  		THEN score
				  	   		ELSE NULL END), 0) AS score_en,
		  	  COALESCE (MAX(CASE WHEN subject = '국어'
				  	   		THEN score
				  	   		ELSE NULL END), 0) AS score_nl,
		  	  COALESCE (MAX(CASE WHEN subject = '수학'
				  	   		THEN score
				  	   		ELSE NULL END), 0) AS score_mt
  	   FROM ScoreRows SR
  	   WHERE SR.student_id = ScoreColsNN.student_id)
 WHERE EXISTS (SELECT *
			   FROM ScoreRows
			   WHERE student_id = ScoreColsNN.student_id)
;
```

- 이들 코드는 두 단계에 걸쳐 NULL 대응을 한다
1. 처음부터 테이블 사이에 일치하지 않는 레코드가 존재한 경우
   - 이러한 레코드는 처음부터 갱신 대상에서 제외해야 한다
   - WHERE 구에 EXISTS를 사용해 '2개의 테이블 사이에 학생 ID가 일치하는 레코드 한정'이라는 조건을 추가했다
2. 학생은 존재하지만 과목이 없는 경우
   - '레코드는 있지만 필드가 없는 경우'를 말한다
   - COALESCE 함수를 적용해 NULL인 경우에는 0을 반환하도록 한다

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/199649676-0d96e65c-4d04-4a75-a9b0-4c05e42574c9.png)

### MERGE 구문 사용
```SQL
MERGE INTO ScoreColsNN
USING (SELECT student_id,
			  COALESCE (MAX(CASE WHEN subject = '영어'
				  	  		THEN score
				  	   		ELSE NULL END), 0) AS score_en,
		  	  COALESCE (MAX(CASE WHEN subject = '국어'
				  	   		THEN score
				  	   		ELSE NULL END), 0) AS score_nl,
		  	  COALESCE (MAX(CASE WHEN subject = '수학'
				  	   		THEN score
				  	   		ELSE NULL END), 0) AS score_mt
  	   FROM ScoreRows
  	   GROUP BY student_id) SR
	ON(ScoreColsNN.student_id = SR.student_id)
WHEN MATCHED THEN 
	UPDATE SET ScoreColsNN.score_en = SR.score_en,
			   ScoreColsNN.score_nl = SR.score_nl,
			   ScoreColsNN.score_mt = SR.score_mt;
```
- MERGE 구문을 사용하면 UPDATE때는 두 개의 장소에 분산되어 있던 결합 조건을 ON구로 한번에 끝낼 수 있다는 것이다
- MERGE는 UPDATE와 INSERT를 한 번에 시행하려고 고안된 기술이지만 UPDATE 또는 INSERT만 수행해도 구문에 이상이 없다는 점을 이용한 트릭이다

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/199649472-56a2b1e9-b44d-416b-9f5c-3e87993c0de2.png)

- ScoreRows 테이블에 풀 스캔 1회 + 정렬 1회 정도가 필요하다
- 이는 갱신할 필드가 많아져도 변하지 않는다
- 상관 서브쿼리를 여러번 사용할 때와 달리 성능이 악화될 염려가 없다
- ScoreColsNN과 결합도 한 번만 있으므로 EXISTS를 사용할 때 보다 더 나을 수 있다