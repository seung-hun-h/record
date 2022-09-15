# 오라클 데이터베이스 링크(Database link)
### 데이터베이스 링크란?
- 데이터베이스 링크란 오라클 데이터베이스와 다른 원격의 데이터베이스를 연결하는 것을 말한다
- 원격의 다른 오라클 데이터베이스나 SQL Server나 MySQL같은 ODBC 호환 데이터베이스도 연결이 가능하다

### 데이터베이스 링크를 사용하는 이유
- 데이터베이스 링크를 사용하면 사용자나 프로그램으로 하여금 한 데이터베이스에서 다른 데이터베이스의 스키마를 참조할 수 있다
- 데이터베이스 스키마를 생성한 이후에는 다음과 같은 방법으로 데이터베이스 링크를 사용할 수 있다

```SQL
SELECT * FROM remote_table@database_link;
```

### 오라클 시노님과 함께 데이터베이스 링크 사용하기
- 오라클 시노님을 사용하면 데이터베이스 링크를 좀 더 간단하게 사용할 수 있다

```SQL
CREATE SYNONYM local_table
FOR remote_table@database_link;
```

- 시노님을 생성한 후에는 다음과 같이 일반 테이블을 사용하듯이 사용할 수 있다

```SQL
SELECT * FROM local_table;
```

### 데이터베이스 링크 생성
- 데이터베이스 링크는 private, public 두 가지가 존재한다
	- private은 데이터베이스 링크 소유자만 접근할 수 있고, public은 모든 사용자가 접근할 수 있다

```SQL
CREATE DATABASE LINK dblink
CONNECT TO remote_user IDENTIFIED BY password
USING 'remote_database';
```
1. `CREATE DATABASE LINK` 뒤에 데이터베이스 링크 이름을 작성한다
2. `CONNECT TO`와 `IDENTIFIED` 뒤에 사용자 이름과 패스워드 정보를 작성한다
3. `USING` 뒤에 원격 데이터베이스의 서비스 이름을 명세한다. 만약 데이터베이스 이름만 작성하면 오라클은 데이터베이스 도메인을 연결 문자열에 추가하여 전체 서비스 이름을 구성한다
   - 일반적으로 `tnsnames.ora` 파일에 항목을 추가하고 `USING` 절에서 remote_database로 참조한다
```SQL
CREATE DATABSE LINK dblink
	CONNECT TO remote_user IDENTIFIED BY password
	USING '(DESCRIPTION=
				(ADDRESS=(PROTOCOL=TCP)(HOST=oracledb.example.com)(PORT=1521))
				(CONNECT_DATA=(SERVICE_NAME=service_name)))';
```

- public 데이터베이스 링크를 만들기 위해서는 `public` 키워드를 추가하면 된다

```SQL
CREATE PUBLIC DATABASE LINK dblink
CONNECT TO remote_user IDENTIFIED BY password
USING 'remote_database';
```

### 데이터베이스 링크 생성 예제
- 원격 데이터베이스의 IP는 `10.50.100.143`이고 포트는 `1521` 서비스 이름은 `SALES`라고 가정한다

1. `tnsnames.ora` 파일에 다음과 같이 작성한다
```TEXT
SALES = 
(DESCRIPTION = 
	(ADDRESS = (PROTOCOL = TCP)(HOST = 10.50.100.143)(PORT = 1521))
	(CONNECT_DATA = 
	(SERVER = DEDICATED)
	(SERVICE_NAME = SALES_PRD)
	)
)
```

2. `CREATE DATABASE LINK` 를 사용해 데이터베이스 링크 를 생성한다

```SQL
CREATEA DATABASE LINK sales
	CONNECT TO bob INDENTIFIED BY abcd1234
	USING 'SALES';
```