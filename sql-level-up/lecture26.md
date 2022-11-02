# 갱신은 효율적으로
---
- SQL이라는 언어는 탄생할 때부터 데이터베이스에서 정보를 검색하는 것을 주 목적으로 설계되었다
- 그러다 보니 자연스럽게 관심이 비교적 적은 갱신은 성능이 좋지 않은 방향으로 작성된다
- 갱신이 비효율적으로 작성된 가장 큰 예시가 '반복계'이다
	- 한 번에 한 개의 레코드를 갱신하는 간단한 SQL 구문을 반복으로 돌린다
- 갱신을 효율적으로 수행하는 SQL을 케이스 스터디로 알아본다


## 1. NULL 채우기
- (keycol, seq)를 키로하는 테이블이있다
- val은 같은 값이 연속되는 경우 생략되어 여러 군데가 비어있다
  
**OmitTbl: 채우기 이전 상태**
![image](https://user-images.githubusercontent.com/60502370/199494412-60d4e6a9-00a8-4afb-a5bd-73ad7028c253.png)

- 이제 NULL이 들어가있는 값에 진짜 값을 넣어본다

**OmitTbl: 값을 채운 뒤 상태**
![image](https://user-images.githubusercontent.com/60502370/199494751-5408a595-e0c6-4de2-87e0-4d1449badf7a.png)

- 호스트 언어를 사용한 '반복계'가 가장 먼저 생각나겠지만 효율적이지 않다
- 계산을 할 때 하나의 레코드만으로는 어렵고 레코드 간 비교가 필요하다
	- 고전적인 SQL의 사고 방식을 사용하면 상관 서브 쿼리가 생각날 것이다
		1. 같은 keycol 필드를 가짐
		2. 현재 레코드보다 작은 seq 필드를 가짐
		3. val 필드가 NULL이 아님

**상관 서브쿼리 사용**
```SQL
UPDATE OmitTbl
SET val = (SELECT val
           FROM OmitTbl OT1
           WHERE OT1.keycol = OmitTbl.keycol
           AND OT1.seq = (SELECT MAX(seq)
                          FROM OmitTbl OT2
                          WHERE OT2.keycol = OmitTbl.keycol
                          AND OT2.seq < OmitTbl.seq
                          AND OT2.val IS NOT NULL))
WHERE val IS NULL
;
```

**실행 계획**
![image](https://user-images.githubusercontent.com/60502370/199495472-d7642e5d-a6b2-4c1a-a521-64ca1864c63b.png)

## 2. 반대로 NULL을 작성
- NULL을 값으로 채우기 이전 상태로 돌리는 쿼리도 이와 비슷하다

```SQL
UPDATE OmitTbl
SET val = CASE WHEN val
                        = (SELECT val
                           FROM OmitTbl OT1
                           WHERE OT1.keycol = OmitTbl.keycol
                           AND OT1.seq = (SELECT MAX(seq)
                                          FROM OmitTbl OT2
                                          WHERE OT2.keycol = OmitTbl.keycol
                                          AND OT2.seq < OmitTbl.seq))
                THEN NUll
                ELSE val END
;
```
- 이러한 코드가 가능한 이유는 서브쿼리가 하나의 값을 리턴하는 스칼라 서브쿼리이기 때문이다.
