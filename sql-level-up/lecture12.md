# 12강 집약
---
- SQL에는 집약 함수(Aggregate function)이라 하는 다른 함수와 구별해서 부르는 함수가 있다
- 집약 함수는 여러 개의 레코드를 한 개의 레코드로 집약하는 기능을 가지고 있다
- 다음 5개의 함수가 집약 함수다
	- COUNT
	- SUM
	- AVG
	- MAX
	- MIN

## 1. 여러 개의 레코드를 한 개의 레코드로 집약
- 예를 들어 다음과 같은 비집약 테이블이 있다고 가정한다

**NonAggTbl**
| id  | data_type | data_1 | data_2 | data_3 | data_4 | data_5 | data_6 |
| --- | --------- | ------ | ------ | ------ | ------ | ------ | ------ |
| Jim | A         | 100    | 10     | 34     | 346    | 54     |        |
| Jim | B         | 45     | 2      | 167    | 77     | 90     | 157    |
| Jim | C         |        | 3      | 687    | 1355   | 324    | 457    |
| ... | ...       | ...    | ...    | ...    | ...    | ...    | ...    |

- 사람을 관리하는 id와 데이터를 종류 별로 관리하는 data_type필드를 기본키로한다
- 업무 A에서는 data_1 ~ data_2, 업무 B에서는 data_3 ~ data_5, 업무 C에서는 data_6을 사용한다고 가정한다
- 현재 비집약 데이터에는 한 사람에 관한 레코드가 분산되어 있다
	- 하지만 이런 데이터를 처리하는 애플리케이션이라면 한 사람에 대한 데이터는 한 개의 레코드로 얻는 것이 편할 것이다

- UNION을 사용하여 레코드를 집약할 수 있다고 생각할 수 있지만 불가능하다
	- 쿼리 결과 필드가 다르다
	- 그리고 UNION으로 쿼리 결과를 머지하는 것은 안티패턴이다

- 이런 데이터는 다음과 같은 집약 테이블(AggTbl)로 만드는 것이 바람직하다
	- 한 사람에 대한 정보가 모두 같은 레코드에 들어있다

| id  | data_1 | data_2 | data_3 | data_4 | data_5 | data_6 |
| --- | ------ | ------ | ------ | ------ | ------ | ------ |
| Jim | 100    | 10     | 167    | 77     | 90     | 457    |
| ... | ...    | ...    | ...    | ...    | ...    | ...    |

### CASE 식과 GROUP BY 응용
- 집약을 위해 GROUP BY를 사용하고 집약 키는 id로 한다
- 선택할 필드를 data_type 필드로 분기한다

```SQL
SELECT id,
	CASE WHEN data_type = 'A' THEN data_1 ELSE NULL END AS data_1,
	CASE WHEN data_type = 'A' THEN data_2 ELSE NULL END AS data_2,
	CASE WHEN data_type = 'B' THEN data_3 ELSE NULL END AS data_3,
	CASE WHEN data_type = 'B' THEN data_4 ELSE NULL END AS data_4,
	CASE WHEN data_type = 'B' THEN data_5 ELSE NULL END AS data_5,
	CASE WHEN data_type = 'C' THEN data_6 ELSE NULL END AS data_6,
FROM NonAggTbl
GROUP BY id;
```

- 위 같은 쿼리를 작성할 수 있다고 생각하겠지만 에러가 난다
- GROUP BY구로 집약 했을 때 SELECT 구에 입력할 수 있는 것은 다음과 같은 세 가지 뿐이다
	- 상수
	- GROUP BY 구에서 사용한 집약 키
	- 집약 함수
- 따라서 다음과 같이 변경해야 한다

```SQL
SELECT id,
	MAX(CASE WHEN data_type = 'A' THEN data_1 ELSE NULL END) AS data_1,
	MAX(CASE WHEN data_type = 'A' THEN data_2 ELSE NULL END) AS data_2,
	MAX(CASE WHEN data_type = 'B' THEN data_3 ELSE NULL END) AS data_3,
	MAX(CASE WHEN data_type = 'B' THEN data_4 ELSE NULL END) AS data_4,
	MAX(CASE WHEN data_type = 'B' THEN data_5 ELSE NULL END) AS data_5,
	MAX(CASE WHEN data_type = 'C' THEN data_6 ELSE NULL END) AS data_6,
FROM NonAggTbl
GROUP BY id;
```

- 해당 쿼리를 사용해 새로운 집약 테이블(AggTbl)이나 뷰를 만들어 사용할 수 있다

### 집약, 해시, 정렬
- 집약  쿼리의 실행 계획을 보면 'HASH GROUP BY'를 확인할 수 있다(오라클)
	- 이는 GROUP BY로 집약 시 해시라는 알고리즘을 사용한다는 의미다
	- 일반적으로는 해시를 사용해 집약하고 가끔식 정렬을 사용하기도 한다
	- 정렬은 'SORT GROUP BY'이다
- 정렬을 사용한 집약은 고전적인 방법이고 최근에는 해시를 사용한다
- 해시의 특성상 GROUP BY의 유일성이 높으면 더 효율적으로 동작한다

#### 성능상 주의점
- 정렬과 해시 모두 메모리를 사용하므로 충분한 워킹 메모리가 확보되지 않으면 스왑이 발생한다
	- 오라클에서는 정렬과 해시를 위해 PGA라는 메모리 영역을 사용한다
- 스왑이 발생하면 저장소 위의 파일이 사용되어 굉장히 느려진다
- 따라서 연산 대상 레코드 수가 많은 GROUP BY구를 사용하는 SQL에서는 충분한 성능 검증을 실행해줘야 한다
- 성능 악화만 발생하면 나름 괜찮은 일이고, 스왑 영역을 모두 사용하면 SQL 구문이 비정상적으로 종료되는 경우도 있다


## 2. 합쳐서 하나
- 생략