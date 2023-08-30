### @ContextConfiguration
- 통합 테스트를 위해 `ApplicationContext`를 구성하고 로드하기 위한 정보들을 정의한다
- 애플리케이션 컨텍스트 정보 지정하는 용도로 사용한다

```Java
@ContextConfiguration(classes = TestConfig.class)
class ConfigClassApplicationContextTests {
	// class body...
}
```

### @SpringJUnitConfig
- `@ExtendWith(SpringExtention.class)` + `@ContextConfiguration`
- `@ExtendWith(SpringExtention.class)`는 JUnit에서 제공하는 애너테이션이다
	- JUnit에 대해 Extension을 등록하기 위한 애너테이션
	- Extension이란 테스트 클래스 혹은 메서드에 대한 행위를 확장하기 위한 것
	- BeforeEach, AfterEach, BeforeAll, AfterAll 모두 Extension에 해당한다
	- `SpringExtension`은 스프링 테스트 프레임워크와 JUnit을 통합하기 위한 Extension

- 결론: 스프링 테스트 프레임워크와 JUnit을 통합하고, 애플리케이션 컨텍스트를 설정하기 위한 애너테이션이다

### @TestConfiguration
- 테스트를 위한 추가 빈이나 커스텀 설정을 추가하기 위해 설정 클래스에 사용하는 애너테이션이다

### @BootStrapWith
- 스프링 테스트 프레임워크를 띄우기 위한 설정을 정의하는 애너테이션
- `@SpringBootTest`애너테이션에는 `@BootstrapWith(SpringBootTestContextBootstrapper.class)` 을 사용사고 있다
- `SpringBootTestContextBootstrapper`는 스프링 테스트 프레임워크를 띄우기 위한 정보를 제공한다

### @SpringBootTest
- 스프링 부트 기반의 테스트를 실행하기 위해 사용하는 애너테이션이다.
- `ContextConfiguration(loader=...` 이 지정되어 있지 않으면 `SpringBootContextLoader`를 기본 `ContextLoader`로 사용한다
- `TestRestTemplate`, `WebTestClient`를 등록한다
- 지정된 포트 혹은 무작위 포트에 바인딩된 테스트용 웹 서버를 제공한다
- 중첩 `@Configuration`과 클래스를 사용하지 않을때 `@SpringBootConfiguration`을 자동으로 찾는다
- 커스텀 properties를 지원한다
  