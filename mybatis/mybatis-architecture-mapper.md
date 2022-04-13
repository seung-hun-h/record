# MyBatis 구조와 매퍼 등록
## MyBatis의 구조
![image](https://user-images.githubusercontent.com/60502370/163106667-65ea707b-fd89-4e25-8634-541592563656.png)

(1) - (3)은 애플리케이션 동작 중 한 번만 실행되고, (4) - (10)은 사용자의 매 요청마다 실행된다.

1. 애플리케이션이 실행되면 `SqlSessionFactoryBuilder`에게 `SqlSessionFactory` 생성을 요청한다
2. `SqlSessionFactoryBuilder`는 Mybatis Config File을 읽고
3. `SqlSessionFactory`를 생성한다
   - `SqlSessionFactory`는 사용자의 매 요청마다 `SqlSession`을 생성하여 요청을 처리한다
4. 클라이언트로부터 DB와 관련된 요청이 들어온다
5. 요청을 처리하기 위해 애플리케이션은 `SqlSessionFactory`에 `SqlSession`생성을 요청한다
6. `SqlSessionFactory`는 `SqlSession`을 생성한다
7. 애플리케이션은 `SqlSession`으로부터 Mapper Interface의 인스턴스(프록시)를 받는다
8. Mapper Interface의 함수를 호출한다
9. Mapper Interface는 실제 작업을 `SqlSession`에 요청한다
10. `SqlSessionFactory`는 Mapper와 연결된 Mapping File을 읽어 작업을 처리하고 결과를 반환한다

### SqlSessionFactoryBuilder
`SqlSessionFactory`를 빌드하기 위한 클래스이다.
```java
SqlSessionFactory build(InputStream inputStream)
SqlSessionFactory build(InputStream inputStream, String environment)
SqlSessionFactory build(InputStream inputStream, Properties properties)
SqlSessionFactory build(InputStream inputStream, String env, Properties props)
SqlSessionFactory build(Reader reader)
SqlSessionFactory build(Reader reader, String environment)
SqlSessionFactory build(Reader reader, Properties properties)
SqlSessionFactory build(Reader reader, String env, Properties props)
SqlSessionFactory build(Configuration config)
```
InputStream이나 Reader 인스턴스를 통해 XML파일을 읽고, 파싱하여 SqlSessionFactory를 생성한다. properties와 environment는 데이터소스와 트랜잭션 관리자를 포함하여 로드할 환경을 판단한다.

```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
        ...
    <dataSource type="POOLED">
        ...
  </environment>
  <environment id="production">
    <transactionManager type="MANAGED">
        ...
    <dataSource type="JNDI">
        ...
  </environment>
</environments>
```

environment 파라미터를 가진 메소드를 호출하면 해당 환경을 위한 설정을 사용할 것이고, 그렇지 않을 경우 위 예제처럼 디폴트 환경을 사용한다(`Configuration`의 기본 생성자로 인스턴스를 생성하면 기본 설정이 다 되어 있다).

properties 파라미터를 가진 메소드를 호출하면 해당 프로퍼티들을 로드해서 설정한다. 프로퍼티들은 XML 파일에 사용되거나 명시할 수 있다. 프로퍼티가 적용되는 우선순위는 아래와 같다

1. properties 엘리먼트에 명시된 속성을 가장 먼저 읽는다
2. properties 엘리먼트의 클래스패스 자원이나 url 속성으로 부터 로드된 속성을 읽는다. 이미 값이 있다면 덮어 쓴다
3. 메소드의 파라미터로 전달된 속성을 읽는다

### SqlSessionFactory
`SqlSessionFactory`는 클라이언트의 요청이 들어올 때 마다 `openSession()`을 호출하여 새로운 `SqlSession` 인스턴스를 생성한다.

```java
SqlSession openSession()
SqlSession openSession(boolean autoCommit)
SqlSession openSession(Connection connection)
SqlSession openSession(TransactionIsolationLevel level)
SqlSession openSession(ExecutorType execType, TransactionIsolationLevel level)
SqlSession openSession(ExecutorType execType)
SqlSession openSession(ExecutorType execType, boolean autoCommit)
SqlSession openSession(ExecutorType execType, Connection connection)
Configuration getConfiguration();
```

Transaction, Connection, ExecutorType 세 가지 조합으로 `SqlSession`을 생성할 수 있다.

autoCommit과 트랜잭션 격리 수준을 설정할 수 있고, 이미 설정된 커넥션에서 생성하거나 `ExecutorType`을 설정할 수 있다.

`ExecutorType`은 Enum으로 다음 3개의 값을 정의 한다

- `ExecutorType.SIMPLE`
  - 구문 실행마다 PreparedStatement를 생성한다
- `ExecutorType.REUSE`
  - PreparedStatement을 재사용한다
- `ExecutorType.BATCH`
  - 모든 update 구문을 배치처리하고 중간에 select가 실행될 경우 필요하다면 경계를 표시한다.

아무 파라미터 없이 `SqlSession`을 생성하는 경우 다음과 같이 설정된다
- 트랜잭션 스코프 시작
- Connection 객체는 현재 활성화된 환경에 의해 설정된 DataSource 인스턴스를 획득
- 트랜잭션 격리 수준은 드라이버나 DataSource가 기본으로 제공하는 수준으로 설정
- `PrepareStatements`는 재사용되지 않고, update는 배치처리 되지 않는다

### SqlSession
`SqlSession`는 구문을 실행하고 트랜잭션을 커밋하거나 롤백 하며 매퍼 인스턴스를 습득하기 위해 필요한 모든 메소드를 가지고 있다.

- SELECT
  - selectOne(), selectList(), selectCursor(), selectMap()
  - selectCursor는() selectList와() 그 개념은 비슷하지만, iterator를 사용해 데이터를 lazily fetch 하는 특징이 있다
- INSERT
  - insert()
- UPDATE
  - update()
- DELETE
  - delete()
- flushStatements()는 배치 수정 구문을 flush한다.
- commit(), rollback()

## Mapper 등록
Mapper를 등록하는 방법으로 XML파일을 작성할 수 있지만, 이보다는 `@MapperScan`을 사용할 것을 권장한다.

### @MapperScan
`@MapperScan`은 마이바티스-스프링 연동 모듈 1.2.0에서 추가된 기능으로 스프링 버전이 3.1 이상이어야 한다. `@MapperScan`의 `annotationClass` 프로퍼티는 검색할 어노테이션을 지정하고, `markerInterface` 프로퍼티는 검색할 상위 인터페이스를 지정한다. 기본 값은 각각 Annotation, Class이며 `basePackage`아래 모든 인터페이스를 Mapper로 등록한다.

Mapper 빈의 이름은 기본적으로 첫 글자를 소문자로 변환한 형태로 사용하고, `org.springframework.stereotype.Component`, `javax.inject.Named`를 사용하면 빈 이름을 직접 등록할 수 있다.

### MapperProxy
인터페이스의 구현체가 없음에도 빈으로 등록될 수 있는 이유는 MyBatis에서 제공하는 `MapperProxy` 덕분이다. MyBatis는 Mapper가 발견된 경우 프록시를 생성해서 빈으로 등록해준다.

<img width="616" alt="image" src="https://user-images.githubusercontent.com/60502370/163120354-4c01bd2e-81cb-4c79-9321-2a5fdf12d5ba.png">

디버깅 모드로 따라가다 보면 결국 `SqlSession`을 통해 작업이 처리되는 것을 알 수 있다.

<img width="563" alt="image" src="https://user-images.githubusercontent.com/60502370/163120686-d23376c4-9ad0-48d9-b181-d9705b7897e5.png">

### @Mapper
`@MapperScan`을 사용하면 basePackage의 하위 인터페이스들은 Mapper로 등록된다. `@Mapper`는 기본적으로 Mapper를 지정해 주기 위한 어노테이션으로 별 다른 기능은 없고, `@MapperSacan`에서 스캔 대상을 지정하는 용도로 사용할 수 있다.

<img width="569" alt="image" src="https://user-images.githubusercontent.com/60502370/163122868-00ebdaee-a402-419a-8125-97c2a7ea9a77.png">

```java
@Configuration
@MapperScan(basePackages = {"com.example.mybatisbasic"},
annotationClass = org.apache.ibatis.annotations.Mapper.class)
public class MyBatisConfig {
// ...
}

```

하지만 MyBatis-Spring-Boot-Start를 사용하는 경우는 다른데, Mapper를 스캔하지 않아도 `@Mapper`를 사용한 인터페이스를 매퍼로 자동 등록해준다.

### @Repository
사내에서 Mapper Interface에 `@Repository`를 사용한 소스코드가 보인다. `@Repository`를 사용한다고 해서 자동으로 Mapper를 등록해주는 것은 아니고, Mapper Scanning을 해주어야 한다.

`@Repository`를 사용하면 데이터베이스와 관련된 Checked Exception을 Spring의 데이터베이스 예외인 UnChecked Exception인 `DataAccessException`으로 변환 해준다는 장점이 있다.

이를 통해 개발자는 데이터베이스에 종속되지 않은 개발을 할 수 있게 된다.

## 참고
- http://mybatis.org/spring/mappers.html#scan
- https://mybatis.org/spring/factorybean.html
- https://mybatis.org/mybatis-3/ko/java-api.html
- https://codingnojam.tistory.com/27
- https://mangkyu.tistory.com/152
- https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/