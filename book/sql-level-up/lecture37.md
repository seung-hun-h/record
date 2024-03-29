# 인덱스를 사용할 수 없는 경우 대처법
---
- 애플리케이션의 설정으로 처리
- 인덱스 온리 스캔

## 1. 외부 설정으로 처리 - 깊고 어두운 강 건너기
### UI 설계로 처리
- 문제가 생기는 쿼리가 실행되지 않게 애플리케이션에서 제한한다

## 2. 외부 설정을 사용한 대처 방법의 주의점
- 성능과 사용성의 트레이드오프를 통해 타협점을 찾는 것이 중요하다
- 외부 설계를 사용한 대처에 실패, 선택률을 낮출 수 없는 경우가 굉장히 많다
	- 이러한 대처는 프로젝트를 시작하는 단계에서 사용자와 합의 해야 한다
- 프로젝트를 시작하는 단계에서는 성능을 잘 고려하지 않는다
	- 테스트 단계에서 치명적인 성능 문제를 발견한 다면 이미 늦은 경우다
	- 이런 상황이 닥치면, 외부 설정 변경을 배제한 채로 시스템을 튜닝해야 한다

## 3. 데이터 마트로 대처
- 외부 설정에 영향을 받지 않는 방법이다
- 데이터 마트 혹은 개요 테이블이란 특정한 쿼리에서 필용한 데이터만을 저장하는, 상대적으로 작은 크기의 테이블을 의미한다
- 접근 대상 테이블의 크기를 작게 해서 I/O 양을 줄이는 것이 데이터 마트의 목적이다


## 4. 데이터 마트를 채택할 시 주의점
### 데이터 신선도
- 데이터 동기 시점의 문제다
- 특정 시점마다 원본 테이블과 데이터 마트의 데이터를 동기화 해주어야 한다
- 이 간극이 짧을 수록 신선한 데이터이다
- 전통적으로 이러한 동기 작업은 야간 배치로 한다
- 신선도가 중요한 데이터라면 이러한 방법을 채택하기 어렵다

### 데이터 마트 크기
- 데이터 마트를 만드는 목적은 테이블의 크기를 작게 해 I/O 양을 줄이는 것이다
- 원래 테이블에서 크기를 딱히 줄일 수 없다면 데이터 마트를 만들어도 빨라지지 않는다

### 데이터 마트 수
- 데이터 마트 수가 많아지면 관리가 어려워진다
- 더이상 사용하지 않는 좀비 마트가 생기거나, 관리가 불가능 해질 수 있다
- 데이터 마트는 애초에 기능 요건에 의해 만들어지는 엔티티가 아니라 ER에도 등장하지 않는다
- 데이터 마트 수가 늘어나면 저장소 용량을 압박하고, 백업 또는 스탭샷을 할 때의 시간이 오래걸릴 수 있다
- 따라서 데이터 마트에 지나치게 의존하는 것은 좋지 않다

### 배치 윈도우
- 배치 윈도우랑 Job Net이 뭐야

## 5. 인덱스 온리 스캔으로 대처

- 인덱스 온리 스캔은 SQL 구문이 접근하려는 대상의 I/O 감소를 목적으로 한다는 점에서 데이터마트와 똑같다
- 인덱스를 사용한 고속화 방법이다
- 조회하고자 하는 모든 필드를 커버링하는 인덱스가 존재하면 인덱스 온리 스캔을 사용할 수 있다
	- 즉, 테이블 접근을 생략하는 방법이다

## 6. 인덱스 온리 스캔의 주의사항
### DBMS에 따라 사용할 수 없는 경우도 있다
- 최근에는 대부분의 DBMS에서 인덱스 온리 스캔을 지원한다
- 오래된 버전에는 안될 수 있으니 주의해야 한다

### 한 개의 인덱스에 포함할 수 있는 필드 수에 제한이 있다
- 인덱스의 크기는 무제한이 아니며, 포함 할 수 있는 필드 수 또는 크기에 제한이 있다
- 인덱스의 크기가 너무 커지면 물리 I/O를 줄이겠다는 당초의 목적이 희미해진다

### 갱신 오버 헤드가 커진다
- 인덱스는 테이블의 갱신 부하를 올리기 마련이다
- 인덱스 온리 스캔을 위한 커버링 인덱스는 성질상 필연적으로 필드 수가 많아 크기가 큰 인덱스가 되기 쉽다
- 검색을 고속으로 만드는 대신 갱신 성능이 떨어지는 트레이드 오프가 발생한다

### 정기적인 인덱스 리빌드가 필요
- 인덱스에만 접근한다는 것은 다시 말해 검색 성능 자체가 인덱스의 크기에 의존한다는 것이다
- 인덱스의 일부만 읽어들이는 레인지 스캔과 달리, Oracle의 INDEX FAST FULL SCAN 등은 인덱스 풀 스캔을 수행한다
- 따라서 검색 성능이 인덱스 크기에 거의 비례해서, 정기적인 모니터링과 리빌드를 운용에 포함시켜야 한다

### SQL 구분에 새로운 필드가 추가된다면 사용할 수 없다
- 애플리케이션 유지 보수 작업에 의해서 새로운 필드가 추가될 수 있다
	- 이렇게되면 인덱스 온리 스캔을 사용할 수 없게된다
- 커버링 인덱스가 SQL 구문에서 사용하는 모든 필드를 커버할 수 없게된 시점에서 더이상 커버링 인덱스가 아니다
	- 이러한 점에서 인덱스 온리 스캔은 일반적인 인덱스에 비해 애플리케이션 유지 보수에 약한 타입의 튜니이라고 볼 수 있다
