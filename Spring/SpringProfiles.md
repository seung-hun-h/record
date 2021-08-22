# Spring Profiles
Profile은 프레임 워크의 핵심적인 기능 중 하나로, 실행되는 환경에 따라서 다른 Bean들을 사용하거나 관리할 수 있다.

## @Profile
SpringBoot에서는 실행 환경별로 분리된 설정 정보를 사용할 수 있도록 `@Profile` 어노테이션을 제공한다. `@Profile`은 `@Component`, `@Configuration`, `@ConfigurationProperties` 어노테이션이 사용된 어느 곳에서나 사용가능하다

### dev
`dev` 프로파일은 개발에서만 해당 컨테이너가 동작하는 것을 의미한다. 즉 `production`에서는 컨테이너가 동작하지 않는다.
```java
@Component
@Profile("dev")
public class DevDatasourceConfig
```
`!`을 사용하여 Not 연산을 할 수도 있다.
```java
@Component
@Profile("!dev")
public class DevDatasourceConfig
```

XML을 통해서 프로파일 설정도 가능하다. `beans` 태그의 `profile` 속성을 사용하면 된다.
```xml
<beans profile="dev">
    <bean id="devDatasourceConfig" 
      class="org.baeldung.profiles.DevDatasourceConfig" />
</beans>
```

## @ActiveProfile
특정 프로파일을 적용하기 위한 어노테이션인 `@ActiveProfile`을 사용하여 테스트에서 쉽게 프로파일을 적용할 수 있다.

## spring.profiles.active
`spring.profiles.active` 프로퍼티를 사용하여 어떤 프로파일을 active할 것인지 설정할 수 있다.

```yaml
spring.profiles.active=dev
```
만약 `spring.profiles.active`에 대한 설정이 없다면 default 설정이 적용되는데 이는 `spring.profiles.default` 프로퍼티로 설정할수 있다.
```yaml
spring.profiles.default=none
```

# Profiles 예제

개발과 프로덕션 환경에서 사용하는 datasource 설정 정보를 분리해야 한다고 가정한다.

두 개의 datasource 구현체들을 추상화한 `DatasourceConfig` 인터페이스를 만든다

```java
public interface DatasourceConfig {
    public void setup();
}
```

개발 환경에서 Configuration 클래스

```java
@Component
@Prifle("dev")
public class DevDatasourceConfig implements DatasourceConfig {
    @Override
    public void setup() {
        System.out.println("Setting up datasource for DEV environment");
    }
}
```

프로덕션 환경에서 Configuration 클래스

```java
@Component
@Profile("Production")
public class ProductionDatasourceConfig implements DatasourceConfig {
    @Override
    public void setup() {
        System.out.println("Setting up datasource for PRODUCTION environment.");
    }
}
```

다음으로 테스트 클래스를 작성해 `DatasourceConfig`의 구현체를 자동 주입 받아보면, 현재 active 상태인 프로파일에 따라 다른 Bean을 주입해줄 것 이다.

```java
public class SpringProfilesWithMavenPropertiesIntegrationTest {
    @Autowired
    DatasourceConfig datasourceConfig;

    public void setupDatasource() {
        datasourceConfig.setup();
    }
}
```

만약 `dev` 프로파일이 active 상태라면 `DevDatasourceConfig` Bean이 주입되었을 것이며 아래와 같은 결과가 출력되었을 것이다.

>Setting up datasource for DEV environment.

# Profiles in Spring Boot
스프링 부트는 프로파일 설정을 편리하게 할 수 있도록 지원한다.

## Programmatically Setting Profiles
프로그래밍적으로 프로파일을 설정하기 위한 방법으로 `SpringApplication` 클래스를 사용할 수 있다.
```java
SpringApplication.setAdditionalProfiles("dev");
```

Maven에서도 프로파일을 설정할 수 있는데 `spring-boot-maven-plugin` 아래에 특정 프로파일 이름을 작성하면된다.
```xml
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <profiles>
                <profile>dev</profile>
            </profiles>
        </configuration>
    </plugin>
    ...
</plugins>
```

## Profile-specific files
스프링 부트에서 프로파일과 관련한 가증 중요한 기능은 **Profile-specific files**이다.  이 파일들은 `application-{profile}`과 같은 네이밍 컨벤션을 지켜야한다. properties를 사용하면 `application-prod.properties`, yaml을 사용하면 `application-prod.yml` 처럼 네이밍한다.<br/>

스프링 부트는 `application.properties` 파일의 위치를 기준으로 다른 Profile-specific 파일들을 로드한다.<br/>

만약 프로덕션 프로파일에서는 MySQL을 사용하고, 개발 프로파일에서는 h2 데이터 베이스를 사용하면 다음과 같이 properties를 작성할 수 있다.

**application-production.properties**
```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/db
spring.datasource.username=root
spring.datasource.password=root
```

**application-dev.properties**
```properties
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
```

## Multi-Document Files
스프링 부트 2.4부터 `application.properties`에서 Multi-Document Files을 작성할 수 있게 기능이 확장되었다. `#---`으로 문서들을 구분한다.
```properties
my.prop=used-always-in-all-profiles
#---
spring.config.activate.on-profile=dev
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/db
spring.datasource.username=root
spring.datasource.password=root
#---
spring.config.activate.on-profile=production
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
```
## Profile Groups
유사한 프로파일들을 그루핑할 수 있도록 Profile Group 기능을 제공한다. 프로덕션 환경을 위한 다수의 프로파일 설정을 그루핑 하기 위해서 아래와 같이 작성할 수 있다.

```properties
spring.profiles.group.production=proddb,prodquartz
```

스프링 부트 2.4 부터 달라지는 프로파일 설정에 관한 자세한 내용은 [여기](http://honeymon.io/tech/2021/01/16/spring-boot-config-data-migration.html)에 정리가 잘되어있다.
