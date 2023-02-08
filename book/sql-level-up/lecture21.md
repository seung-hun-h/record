# 21강 서브쿼리가 일으키는 폐해
---
## 1. 서브쿼리의 문제점
서브쿼리의 문제점은 서브쿼리가 실체적인데이터를 가지지 않는다는 점에서 기인한다

- 연산 비용 추가
	- 서브쿼리에 접근할 때마다 SELECT 구문을 실행해서 데이터를 만들어야 한다
	- 따라서 SELECT 구문을 실행하는 비용이 매번 발생한다
- 데이터 I/O 비용 발생
	- 연산의 결과를 어딘가에 저장해야 한다
	- 데이터가 크면 파일에 결과를 쓰는 경우가 있으므로 비용이 커질 수 있다
- 최적화를 받을 수 없음
	- 서브쿼리는 일반 테이블과 사용하는 방법은 차이가 없지만, 서브쿼리에는 메타 정보가 하나도 존재하지 않는다
	- 옵티마이저가 쿼리를 해석하기 위해 필요한 정보를 서브쿼리에서는 얻을 수 없다

## 2. 서브쿼리 의존증
- 고객의 구입 명세 정보를 기록하는 Receipts 테이블이 있다
- 고객들이 구매했던 가장 오래된 구매이력 찾기

**Receipts**
![image](https://user-images.githubusercontent.com/60502370/196297478-44849267-08ee-46dc-9847-561295d18fad.png)

**기대 결과**
![image](https://user-images.githubusercontent.com/60502370/196297633-2f2cbe89-2b89-4526-b105-8350f5ec497e.png)


### 서브쿼리를 사용하는 방법
```SQL
SELECT R1.cust_id, R2.min_seq, R1.price
FROM Receipts R1 
    INNER JOIN 
        (SELECT cust_id, MIN(seq) AS min_seq
            FROM Receipts
            GROUP BY cust_id) R2
        ON R1.cust_id = R2.cust_id        
        AND R1.seq = R2.min_seq
;
```

- 장점
	- 간단하다
- 단점
	- 코드가 복잡해서 읽기 어렵다. 서브 쿼리를 사용하면 코드가 여러 계층에 걸쳐 만들어진다
	- 성능이 좋지 않다
	  1. 서브쿼리는 대부분 일시적인 영역에 확보되므로 오버헤드가 생긴다
	  2. 서브쿼리는 인덱스 또는 제약 정보를 가지지 않기 떄문에 최적화되지 못한다
	  3. 이 쿼리는 결합을 필요로 하기 때문에 비용이 높고 실행 계획 변동 리스크가 발생한다
	  4. Receipts 테이블에 스캔이 두번 필요하다

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/196298129-25a59547-b43b-45c8-8aea-8a9312c6d363.png)
- 실행 계획을 보면 R1, R2 테이블에 대한 해시 결합이 발생했다
- 책에서는 Receipts 테이블에 2번 접근했지만, 실습해보면 Receipts 테이블 풀 스캔 + 인덱스 풀 스캔이 발생했다
- 결합 제거와 Receipts 테이블에 한 번만 접근하는 방식으로 성능을 높일 수 있을 것 같다

### 상관 서브 쿼리는 답이 될 수 없다
- 상관 서브 쿼리를 사용한 동치 변환 방법을 쉽게 떠올릴 수 있지만, 이는 정답이 아니다

```SQL
SELECT cust_id, seq, price
FROM Receipts R1
WHERE seq = (SELECT MIN(seq)
             FROM Receipts R2
             WHERE R1.cust_id = R2.cust_id);
```

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/196299614-4d47e056-18ca-40ea-8dcb-e98b6630805f.png)
- 책에서는 테이블 풀 스캔이 2회 발생했지만, 실습에서는 테이블 풀 스캔 + 인덱스 레인지 스캔이 발생했다
- 단순 서브 쿼리를 사용할 때(인덱스 풀 스캔) 보다는 조금 더 나아졌지만 여전히 부족하다

### 윈도우 함수로 결합 제거
- SQL 튜닝에서 가장 중요한 부분이 I/O를 줄이는 것이다
- 접근을 줄이려면 윈도우 함수 `ROW_NUMBER`를 다음과 같은 형태로 사용한다

```SQL
SELECT cust_id, seq, price
FROM (SELECT cust_id, seq, price,
        ROW_NUMBER()
            OVER(PARTITION BY cust_id ORDER BY seq) AS row_seq
      FROM Receipts) work
WHERE work.row_seq = 1
;      
```

- seq 값이 불확실해 쿼리를 한 번 더 사용해야 했던 이전의 문제가 해결되었다
- 쿼리도 간단해졌고 가독성도 올라 갔다

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/196300303-164aea75-37db-4bc5-bf81-d89808bcf701.png)

- 윈도우 함수를 사용해 정렬 작업이 추가되기는 했지만, 이전의 쿼리들의 MIN을 사용했으므로 이런 부분에서는 큰 비용을 차지 하지 않는다

## 3. 장기적인 관점에서의 리스크 관리
- 결합이나 상관 서브 쿼리를 사용한 것에 비해 윈도우 함수를 사용한 쿼리가 얼마나 성능이 더 좋은지는 여러 요인에 의해 크게 바뀔 수 있다
- 하지만 SQL 튜닝의 기본은 I/O를 줄이는 것이다
- 결합을 제거하면 성능 향상뿐 아니라 성능의 안정성도 확보할 수 있다
- 결합의 불안정 요소 
	- 결합 알고리즘의 변동 리스크
	- 환경 요인에 의한 지연 리스크

### 알고리즘 변동 리스크
- 결합 알고리즘에는 Nested Loops, Short Merge, Hash 세 가지가 있다
	- 옵티마이저가 테이블의 크기등을 고려해 자동으로 결정한다
- 대략적으로 테이블의 크기가 작은 경우 Nested Loops를 사용하고, 큰 경우 Short Merge나 Hash를 사용한다
- 하지만 시스템을 운영하면서 레코드가 많아지면 실행 계획에 변동이 생기고 성능에 큰 변화가 일어난다
	- Short Merge, Hash는 메모리 사용양이 많아지면 TEMP 탈락 현상을 발생시켜 성능을 크게 저하한다

### 환경 요인에 의한 지연 리스크
- Nested Loops의 내부 테이블 결합 키에 인덱스가 존재하면 성능이 크게 향상된다
- Short Merge나 Hash를 사용해 TEMP 탈락이 발생할때 메모리를 늘려주면 해결될 수 있다
- 하지만 항상 결합 키에 인덱스가 존재하는 것이 아니고, 메모리 튜닝은 한정된 리소스 내부에서 트레이드오프를 발생시킨다
- 결합을 사용한다는 것은 장기적인 관점에서 고려해야 할 리스크를 늘리게 된다는 뜻이다
- 다음 사항을 꼭 기억하자
	- 실행 계획이 단순할수록 성능이 안정적이다
	- 엔지니어는 기능(결과)뿐 아니라 비기능적인 부분(성능)도 보장할 책임이 있다

## 4. 서브쿼리 의존증 - 응용편
- 이제는 Receipts 테이블에서 최소 순번과 최대 순번의 가격 차를 구해 본다

**예상 결과**
![image](https://user-images.githubusercontent.com/60502370/196359783-62b29a2f-0c3b-4c95-b307-d6ab8eb1889f.png)

### 다시 서브쿼리 의존증
```SQL

SELECT TMP_MIN.cust_id,
    TMP_MIN.price - TMP_MAX.price AS diff
FROM (SELECT R1.cust_id, R1.seq, R1.price
      FROM Receipts R1
        INNER JOIN 
            (SELECT cust_id, MIN(seq) AS min_seq
             FROM Receipts
             GROUP BY cust_id) R2
        ON R1.cust_id = R2.cust_id
        AND R1.seq = R2.min_seq) TMP_MIN
    INNER JOIN
        (SELECT R3.cust_id, R3.seq, R3.price
          FROM Receipts R3
            INNER JOIN 
                (SELECT cust_id, MAX(seq) AS min_seq
                 FROM Receipts
                 GROUP BY cust_id) R4
            ON R3.cust_id = R4.cust_id
            AND R3.seq = R4.min_seq) TMP_MAX
    ON TMP_MIN.cust_id = TMP_MAX.cust_id;
```

- 최솟값의 집합을 찾고 최댓값의 집합을 찾은 후 고객 ID를 키로 결합하면 된다
- 쿼리가 길어졌고 서브쿼리의 계층이 깊어졌다
- 테이블에 대한 접근도 2배 증가해 4번이 됐다

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/196360418-db830f03-abfe-4b78-a040-43f4d1c4823f.png)
- 실습해보면 테이블 풀 스캔 + 인덱스 풀 스캔이 2번 발생했다

### 레코드 간 비교에서도 결합은 불필요
```SQL
SELECT cust_id,
        SUM(CASE WHEN work.min_seq = 1 THEN price ELSE 0 END)
        - SUM(CASE WHEN work.max_seq = 1 THEN price ELSE 0 END) AS diff
FROM (SELECT cust_id, price,
        ROW_NUMBER() OVER(PARTITION BY cust_id ORDER BY seq) AS min_seq,
        ROW_NUMBER() OVER(PARTITION BY cust_id ORDER BY seq DESC) AS max_seq
      FROM Receipts) work
WHERE work.min_seq = 1
    OR work.max_seq = 1
GROUP BY cust_Id
```

- 레코드간 비교에서도 결합 없이 윈도우 함수로 해결할 수 있다
- 이렇게하면 서브 쿼리는 `work` 하나면 충분하다

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/196364464-061ba09b-0479-4e2f-af62-81cb2aed1109.png)
- Receipts 테이블에 대한 접근이 1회로 줄었다
- 정렬이 2회 발생하지만 결합을 반복하는 것보다 저렴하고 실행계획의 안정성도 확보할 수 있어 괜찮은 거래라고 볼 수 있다

## 5. 서브쿼리는 정말 나쁠까?
- 서브쿼리 자체가 나쁜것은 아니다
- 결과적으로 서브쿼리를 뺴는 것이 좋아도, 처음 쿼리를 고민할 때는 먼저 서브쿼리를 사용하는 쪽으로 풀어보는 것이 이해가 쉽다
- 즉, 서브쿼리는 생각의 보조 도구가 될 수 있다
	- 각 부분을 조합해서 최종 결과를 만드는 바텀업 방식이다
- 다만 바텀업과 비절차 지향형 언어인 SQL은 지향하는 방향과 성질이 서로 맞지 않다