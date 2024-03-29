# 1강 DBMS 아키텍처 개요
---
- DBMS로는 Oracle, PostgreSQL, MySQL, DB2, SQL Server 등 다양하게 존재한다
- 이러한 DBMS는 상세한 아키텍처는 다르지만 RDB로써 기능을 제공한다는 공통 목표를 가진다
- 따라서 이들은 관계 모델이라는 수학적인 이론을 바탕으로 한다

## DMBS 아키텍처
![image](https://user-images.githubusercontent.com/60502370/190883963-2fa1f09b-2bcd-4e0f-8d7e-b1955e0cf899.png)

### 쿼리 평가 엔진
- 사용자로부터 받은 SQL 구문을 분석하고, 어떤 순서로 기억장치의 데이터에 접근할지를 결정한다
- 이때 결정되는 계획을 실행 계획이라고 한다
	- 실행 계획에 기반하여 데이터에 접근하는 방법을 접근 메서드(access method)라 한다

### 버퍼 매니저
- DBMS는 버퍼라는 특별한 용도의 메모리 영역을 확보해둔다
- 버퍼 매니저는 버퍼를 관리하는 역할을 한다
- 버퍼 매니저는 디스크 용량 매니저와 연동되어 작동한다

### 디스크 용량 매니저
- 디스크 용량 매니저는 어디에 어떻게 데이터를 저장할 지를 관리하며, 데이터의 읽고 쓰기를 제어한다

### 트랜잭션 매니저와 락 매니저
- 여러 사용자가 동시에 DBMS에 접근해서 사용할 때, 각각의 처리는 DBMS 내부에서 트랜잭션이라는 단위로 처리된다
- 트랜잭션의 정합성을 유지시키면서 실행시키고, 필요한 경우 데이터에 락을 걸어 다른 사람의 요청을 대기시키는 것이 트랜잭션 매니저와 락 매니저의 역할이다

### 리커버리 매니저
- 리커버리 매니저는 데이터를 정기적으로 백업하고 문제가 일어났을 때 복구하는 역할을 수행한다