# 8. 테이블과 뷰
## 8.1 테이블
### 8.1.1 테이블 만들기
- 생략

### 8.1.2 제약 조건
- 제약 조건이란 데이터의 무결성을 지키기 위한 제한된 조건을 의미한다
	- 어떠한 조건을 만족했을 때 입력되도록 제약할 수 있다
- 대부분의 DBMS는 데이터의 무결성을 위해서 6가지 제약조건을 제공한다
	- PRIMARY KEY 
	- FOREIGN KEY
	- UNIQUE
	- CHECK
	- DEFAULT
	- NULL

**기본 키 제약 조건**
- 기본 키란 테이블에 존재하는 많은 행을 구분할 수 있는 식별자를 의미한다
- 기본 키는 중복될 수 없고 NULL을 허용하지 않는다
- 기본 키로 생성한 것은 자동으로 인덱스가 생성된다
- 기본 키는 하나 이상의 열로 구성된다
- 기본 키를 설정할 수 있는 방법은 아래와 같다

```SQL
CREATE TABLE user
(
	userID CAHR(8) NOT NULL PRIMARY KEY,
	userName NVARCHAR2(10) NOT NULL

...
```

```SQL
CREATE TABLE user
(
	userID CAHR(8) NOT NULL CONSTRAINT PK_user_userID PRIMARY KEY,
	userName NVARCHAR2(10) NOT NULL


```

```SQL
CREATE TABLE user
(
	userID CAHR(8) NOT NULL,
	userName NVARCHAR2(10) NOT NULL,
	...
	CONSTRAINT PK_user_userID PRIMARY KEY (userID)

	...
```

```SQL
ALTER TABLE user
ADD CONSTRAINT PK_user_userID
PRIMARY KEY (userID)
;
```

**외래 키 제약 조건**
- 외래 키 제약 조건은 두 테이블 사이의 관계를 선언함으로써, 데이터의 무결성을 보장해주는 역할을 한다
- 외래 키 관계를 설정하면 하나의 테이블이 다른 하나의 테이블에 의존하게 된다
- 외래 키를 정의하는 테이블을 '외래 키 테이블'이라 하고 참조 대상 테이블을 '기준 테이블'이라 한다
- 외래 키 테이블에 데이터를 입력할 떄는 반드시 기준 테이블을 참조해야 하므로 기준 테이블에는 이미 데이터가 존재해야 한다
- 기준 테이블의 열은 반드시 기본 키 이거나 UNIQUE 제약 조건을 만족해야한다

```SQL
CREATE TABLE user
(
	userID CHAR(8) NOT NULL PRIMARY KEY
	...
)

CREATE TABLE buy
(
	idNum NUMBER(8) NOT NULL PRIMARY KEY,
	userID CHAR(8) NOT NULL REFERENCES user(userID)
	...
)
```

```SQL
CREATE TABLE user
(
	userID CHAR(8) NOT NULL PRIMARY KEY
	...
)

CREATE TABLE buy
(
	idNum NUMBER(8) NOT NULL PRIMARY KEY,
	userID CHAR(8) NOT NULL CONSTRAINT FK_user_userID REFERENCES user(userID),
	...
)
```

```SQL
CREATE TABLE user
(
	userID CHAR(8) NOT NULL PRIMARY KEY
	...
)

CREATE TABLE buy
(
	idNum NUMBER(8) NOT NULL PRIMARY KEY,
	userID CHAR(8) NOT NULL,
	...
	CONSTRAINT FK_user_userID REFERENCES user(userID)
)
```

```SQL
ALTER TABLE buy
ADD CONSTRAINT FK_user_userID
FOREIGN KEY (userID)
REFERENCES user(userID);
```
- ON DELETE CASCADE 옵션은 기존 테이블의 데이터가 삭제되었을 때, 외래 키 테이블의 데이터도 자동으로 삭제되도록 설정해준다

**UNIQUE 제약 조건**
- 중복되지 않는 유일한 값을 입력해야 하는 제약 조건이다

```SQL
CREATE TABLE user
(
	...
	email CHAR(30) NOT NULL UNIQUE
	...
)
```

**CHECK 제약 조건**
- 입력 되는 데이터를 점검하는 기능을 한다

```SQL
ALTER TABLE user
ADD CONSTRAINT CK_height
CHECK (height >= 0);
```
- ENABLE NOVALIDATE 옵션은 기존에 입력된 데이터가 CHECK 제약 조건에 맞지 않을 경우 무시하고 넘어간다

**DEFAULT 정의**
- 값을 입력하지 않았을 때, 자동으로 입력되는 기본 값을 정의하는 방법이다

**NULL 값 허용**
- NULL 값을 허용하는 지 결정한다

### 8.1.3 임시 테이블
- 임시로 잠깐 사용되는 테이블이다
- 임시 테이블의 데이터는 세션 내에서만 존재하며, 세션이 닫히면 자동으로 데이터가 삭제된다

### 8.1.4 테이블 삭제
- 생략
### 8.1.5 테이블 수정
- 생략

## 8.2 뷰
- 뷰는 일반 사용자 입장에서는 테이블과 동일하게 사용하는 개체이다

### 8.2.1 뷰의 개념
- 뷰는 일반 사용자 입장에서는 테이블과 동일하게 사용하는 개체이다
- 뷰를 생성한 후에는 생성한 뷰를 그냥 테이블처럼 생각하고 접근하니 원래 테이블에 접근한 것과 동일한 결과를 얻을 수 있다
- 뷰는 일반적으로 읽기 전용으로 많이 사용되지만, 뷰를 통해서 테이블의 데이터를 수정하는 것도 가능하다

### 8.2.2 뷰의 장점
**보안에 도움이 된다**
- 사용자에 따라 민감 정보에 접근을 제한하고 싶을 때 뷰를 만들고, 뷰에만 접근 권한을 부여할 수 있다

**복잡한 쿼리를 단순화시켜 줄 수 있다**
- 아래와 같은 복잡한 쿼리를 자주 사용한다면 뷰로 만들어 쿼리를 단순화 할  수 있다
```SQL
SELECT u.userID, u.userName, b.prodName, u.addr, u.mobile1 || u.mobile2
FROM user u
	INNER JOIN buy b
	ON u.userID = b.userID;
```

```SQL
CREATE OR REPLACE VIEW v_userbuy
AS
	SELECT u.userID, u.userName, b.prodName, u.addr, u.mobile1 || u.mobile2
	FROM user u
		INNER JOIN buy b
		ON u.userID = b.userID;
```

### 8.2.3 구체화된 뷰
- 구체화된 뷰(Materialized View)는 실제 데이터가 존재하는 뷰로 정의할 수 있다
- 일반적으로 뷰는 원본 테이블의 데이터를 참조하고 있는데 이 경우 원하는 성능이 나오지 않을 수 있다
- 뷰에 실제 데이터를 넣어 놓는다면 쿼리의 성능이 월등히 향상될 수 있을 것이다
- 구체화된 뷰를 사용하기 적합한 경우는 조회할 때마다 많은 계산 비용이 드는 집계함수나 여러 테이블을 조인하느 결과 집합에 적합하다
- 실제 테이블의 데이터가 변경된다면 그 테이블의 데이터를 원본으로 하는 구체화된 뷰도 변경이 되어야할 지 결정해야 한다
	- 뷰의 내용의 거의 실시간으로 자주 업데이트 된다면 시스템에 부하가 커질 것이다

```SQL
CREATE MATERIALIZED VIEW 뷰이름
[ BUILD { IMMEDIATE | DEFERRED }
  REFRESH { ON COMMIT | ON DEMAND } { FAST | COMPLETE | FORCE | NEVER }
  ENABLE QUERY REWRITE
]

AS
	SELECT 문장;
```

- BUILD IMMEDIATE
	- 구체화된 뷰를 생성한 후 동시에 구체화된 내부에 데이터가 채워진다
- BUILD DEFERRED
	- 구체화된 뷰 내부에 데이터가 나중에 채워진다
- REFRESH ON COMMIT
	- 원본 테이블에 커밋이 될 때마다 구체화된 뷰의 내용이 변경된다
- RECRESH ON DEMAND
	- 직접 DBMS_MVIEW 패키지를 실행해서 구체화된 뷰의 내용을 변경한다
- FAST, FORCE
	- 원본 테이블에 변경된 데이터만 구체화된 뷰에 적용
- COMPLETE
	- 원본 테이블이 변경되면 전체를 구체화된 뷰에 적용시킨다
- NEVER
	- 원본 테이블이 변경되어도 구체화된 뷰에는 적용시키지 않는다
- 어떤 SQL문이 지금 생성하는 구체화된 뷰를 사용하는 것이 성능이 더 좋을 경우에 ENABLE QUERY REWRITE 옵섭을 설정하면 내부적으로 구체화된 뷰를 사용하도록 허용할 수 있다