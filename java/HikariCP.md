## 데이터베이스 커넥션 풀이란
- 데이터베이스와의 커넥션을 미리 생성해서 필요할 때마다 꺼내 사용할 수 있도록 하는 라이브러리 혹은 프레임워크
- 운영하고 있는 애플리케이션의 특성에 맞게 설정을 조정해야 한다

## HikariCP
- Java 언어로 작성된 데이터베이스 커넥션 풀 라이브러리


### Configuration
- Essential Options를 제외한 나머지는 기본 설정이 존재한다

#### Essential 
- `dataSourceClassName`
	- JDBC 드라이버가 제공하는 `DataSource` 클래스 이름
	- `jdbcUrl` 설정이 존재하면 `dataSourceClassName`은 필요 없다

> JDBC Driver 
<img width="678" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/f995e3aa-8268-4273-b028-2d47868df6f7">

- `jdbcUrl`
	- DriverManager 기반으로 HikariCP를 설정할 때 사용한다
	- DriverManager는 JDBD API 중 하나이다. 데이터베이스 연결을 관리한다
	- `driverClassName`도 설정해야 한다. 일단 `driverClassName`을 설정 하지 않고 실행해보고 안되면 그때 실행 하는 것을 권장한다.

- `username`
- `password`

#### Frequently
- `autoCommit`
	- 커넥션 풀의 커넥션들에 대한 auto commit을 설정
	- 기본값: true
- `connectionTimeout`
	- 클라이언트가 커넥션 풀의 커넥션을 획득하는데 기다리는 시간
	- 기본값: 30000(30초)
- `idleTimeout`
	- 커넥션이 커넥션 풀에서 유휴 상태로 유지될 수 있는 최대 시간
	- `minimumIdle`이 `maximumPoolSize`보다 작게 설정되어 있어야 한다
	- 커넥션 풀의 유휴 커넥션이 `minimumIdle`에 도달 하자마자 유휴 커넥션들이 제거되는 것은 아니다. 15~30초 정도는 유지된다
	- `idelTimeout`을 설정하면 최소한 설정한 시간전에는 제거되지 않는다
	- `0`은 유휴 커넥션이 커넥션 풀에서 제거되지 않음을 의미한다
	- 기본값: 600000(10분)
- `keepaliveTimeout`
	- 커넥션을 유지할 수 있는 시간
	- 유휴 커넥션에서만 **keepalive** 가 수행된다
	- `maxLifeTime`보다 작아야 한다
	- **keepalive** 시간에 도달하면 JDBC4는 `isValid()`, 그 외는 `connectionTestQuery()`를 호출한다
	- 기본값: 0
- `maxLifetime`
	- 커넥션 풀의 커넥션에 대한 최대 수명
	- 사용 중인 커넥션은 절대 제거되지 않고, 닫힐 때만 제거된다
	- 데이터베이스나 인프라에서 설정한 connection time out보다 조금 짧아야한다. 이 값을 설정하는 것을 권장한다
	- 0은 최대 수명이 없으나 `idelTimeout` 설정에 따라 제한된다
	- 기본값: 1800000(30분)
- `connectionTestQuery`
	- JDBC4를 사용한다면 이 값을 설정하지 않는다
	- JDBC4를 지원하지 않는 레거시 시스템을 위한 설정이다
	- 데이터베이스에 커넥션이 살아있다는 것을 알리기 위해 사용한다
- `minimumIdle`
	- 커넥션 풀에서 유지되는 최소 유휴 커넥션의 수
	- 유휴 커넥션의 수가 `minimumIdle` 보다 적고 커넥션 풀의 커넥션이 `maximumPoolSize`보다 적을 경우 HikariCP는 최대한 빠른 시간내에 추가 커넥션을 생성하도록 노력한다
	- 애플리케이션이 최대한의 반응성을 제공하기 위해서는 이 값을 설정하지 않는 것을 권장한다
	- 기본값: maximumPoolSize와 동일한 값
- `maximumPoolSize`
	- 유휴 커넥션과 사용중인 커넥션을 포함한 커넥션 풀의 크기
	- 커넥션 풀의 크기가 `maximumPoolSize`에 도달하고, 사용가능한 유휴 커넥션이 없는 경우 `getConnection()`이 `connectionTimeout` 시간동안 block 된다
- `poolName`
	- 커넥션 풀의 이름

#### Infrequenlty
- `initializationFailTimeout`
	- 
- `isolateInternalQueries`
- `allowPoolSuspension`
- `readOnly`
- `RegisterMbeans`
- `catalog`
- `connetionInitSql`
- `driverClassName`
- `validationTimeout`
- `leakDetectionThreshold`
- `dataSource`
- `schema`
- `threadFactory`
- `scheduledExecutor`

### HikariCP 코드
#### HikariDataSource
#### HikariPool


