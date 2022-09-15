# 오라클 시노님(Synonym)

### 시노님이란?
- 시노님(Synonim)은 오라클에서 제공하는 기능으로 스키마 오브젝트에 대한 별칭(Alias)으로 볼 수 있다
	- 스키마 오브젝트란 테이블, 뷰, 시퀀스, 프로시저 등을 말한다

### 시노님를 사용하는 이유
- 테이블이나 뷰 같은 스키마 오브젝트의 이름과 소유자를 숨김으로써 보안 수준을 제공한다
- 물리적으로 분산된 데이터베이스의 위치 투명성을 제공한다
- 다른 유저의 스키마 오브젝트를 동의어를 통해 참조할 수 있다

### 시노님 생성

```SQL
CREATE [OR REPLACE] [PUBLIC | PRIVATE] SYNONYM schema.synonym_name(시노님 이름)
FOR schema.object(스키마 오브젝트);
```

```SQL
CREATE OR REPLACE PUBLIC SYNONYM EX_SYNONYM
FOR CUSTOMERS;
```

1. 먼저 시노님을 생성할 스키마와 시노님 이름을 작성한다 > 스키마.시노님이름
   스키마를 명시하지 않으면 현재 본인이 소유중인 스키마를 기본으로 생성한다
2. `FOR` 이후에 시노님을 생성 하고자 하는 스키마 오브젝트를 명시한다
3. `OR REPLACE` 키워드가 있으면 동일한 시노님이 있는 경우 대체한다. 동일한 시노님이 없는 경우 새로 생성한다
4. 시노님을 `PUBLIC`으로 생성하면 모든 사용자들이 접근할 수 있다. 그렇지 않으면 `PRIVATE`은 시노님을 소유중인 사용자만 접근할 수 있다

- 스키마 오브젝트의 시노님을 만들어 놓으면 INSERT, UPDATE, DELETE, SELECT 같은 SQL 문으로 원 테이블을 참조할 수 있다

### 시노님 삭제
```SQL
DROP SYNONYM schema.synonym_name FORCE;
```

1. `DROP SYNONYM` 뒤에 삭제하고자 하는 시노님 이름을 작성한다. 스키마 이름을 생략하는 경우 현재 스키마의 시노님을 삭제한다
2. `FORCE` 를 사용하여 종속 테이블 또는 사용자 정의 유형이 있는 경우에도 시노님을 삭제한다

- `PUBLIC` 시노님을 삭제하기 위해서는 아래와 같이 SQL문을 작성한다

```SQL
DROP PUBLIC SYNONYM synonym_name FORCE;
```

### 시노님 조회
- 현재 스키마의 시노님 조회
```SQL
SELECT * FROM USER_SYNONYMS;  
```
- 전체 스키마의 시노님 조회
```SQL
SELECT * FROM ALL_SYNONYMS;
```

### 시노님 사용 권한 부여
```SQL
GRANT [SELECT, INSERT, DELETE, UPDATE] ON schema1.synonym_name TO schema2
```

### 시노님의 장점
1. 복잡하고 긴 이름을 간단한 별칭으로 참조할 수 있다
   - `human_resources.employee_locations` 을 `offices` 처럼 간단한 이름으로 사용할 수 있다
2. 레거시 애플리케이션의 하위 호환성을 도울 수 있다
   - 테이블의 이름을 변경하려 할 때 기존의 테이블 이름으로 시노님을 만들고 원 테이블의 이름을 변경하면 현재 애플리케이션에 영향을 주지 않는다
3. 스키마간 오브젝트를 이동할 때 현재 코드를 변경하지 않도록 할 수 있다




