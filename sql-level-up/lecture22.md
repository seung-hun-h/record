# 22강 서브쿼리 사용이 더 나은 경우
---
- 결합할 떄는 최대한 결합 대상 레코드 수를 줄이는 것이 중요하다
- 옵티마이저가 이런것을 잘 판별하지 못할 때는 사람이 직접 연산 순서를 명시해주면 성능적으로 좋은 결과를 얻을 수 있다

## 1. 결합과 집약 순서

**Companies**
![image](https://user-images.githubusercontent.com/60502370/196594443-30e078cc-78e3-493e-8cd0-fe099da762c5.png)

**Shops**
![image](https://user-images.githubusercontent.com/60502370/196594486-9a72a06d-38d2-49c3-af67-4b08ea81824d.png)

**기대 결과**
![image](https://user-images.githubusercontent.com/60502370/196594578-732654e4-8b0e-4bc5-b973-8dce6fe2c022.png)

- 각 회사의 주요 사업소(main_flg가 'Y')의 종업원 수를 구하고, 위와 같은 결과를 도출한다

### 두 가지 방법
1. 결합부터 하고 집약을 하는 방법
```SQL
SELECT c.co_cd,
       MAX(c.district),
       SUM(s.emp_nbr) 
FROM Companies c
    INNER JOIN Shops s
    ON c.co_cd = s.co_cd
WHERE s.main_flg = 'Y'
GROUP BY c.co_cd
```

2. 집약을 먼저하고 결합하는 방법
```SQL
SELECT c.co_cd,
       c.district,
       sum_emp 
FROM Companies c
    INNER JOIN 
        (SELECT co_cd, SUM(emp_nbr) AS sum_emp
         FROM Shops
         WHERE main_flg = 'Y'
         GROUP BY co_cd) csum
    ON c.co_cd = csum.co_cd
;
```

- 첫 번째 방법은 회사 테이블과 사업소 테이블의 결합을 먼저 수행하고 결과에 GROUP BY를 적용했다
- 두 번째 방법은 먼저 사업소 테이블을 집약해서 직원 수를 구하고 회사 테이블과 결합했다


### 결합 대상 레코드 수
- 첫 번째 방법
	- 회사 테이블: 4개
	- 사업소 테이블: 10개
- 두 번째 방법
	- 회사 테이블: 4개
	- 사업소 테이블: 4개
- 중요한 것은 csum 테이블의 레코드 수가 4개로 줄어들었다는 것이다
- 회사 테이블에 비해 사업소 테이블의 규모가 매우 크다면 일단 결합 대상 레코드 수를 집약하는 편이 I/O 비용을 줄일 수 있다
	- 두 번째 방법은 집약 비용이 더 크겠지만 TEMP 탈락이 발생하지 않는다면 괜찮다
- 첫 번째 방법과 두 번째 방법 중 어느쪽이 빠른지는 환경에도 의존한다
	- 따라서 실제 개발을 할 때는 이러한 요인을 모두 고려해서 성능을 테스트하고 판단을 내리는 것이 좋다
- 튜닝 선택지 중 하나로 '사전에 결합 레코드 수를 압축한다'라는 방법이있다는 것은 알아두자