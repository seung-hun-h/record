# Java Logging Framework

## Good Logging
1. 좋은 로그 메시지는 어플리케이션이 내부적으로 동작하는 것을 이해할 수 있는 정보를 제공해야 한다
2. 어플리케이션의 성능에 영향을 미치지 않는 선에서 효과적으로 로그 메시지를 남겨야 한다
3. 배포 환경과 상황에 따라 다르게 로그 메시지를 남길 수 있어야 한다

최근 Logging Framework는 2, 3번에 대한 기술적인 요구 사항은 해결한 상태이다. 따라서 개발자는 본인에게 맞는 적절한 Logging Framework를 선택하고 로그 메시지를 남기기만 하면된다.

## Logging Framework

### SLF4J
SLF4J(Simple Logging Facade for Java)는 다양한 Logging Framework의 추상체나 간단한 퍼사드를 제공하여 최종 사용자가 배포 시 원하는 Logging Framework를 바인딩할 수 있도록 한다.

따라서 SLF4J를 사용하면 구현체의 종류와 상관 없이 일관된 로깅 코드를 작성할 수 있고, Logging Framework를 변경할 경우 최소한의 수정으로 구현체를 변경할 수 있다.

![image](https://user-images.githubusercontent.com/60502370/163738674-780e8595-2037-4f0b-b260-0c014f801b2e.png)

### Logback
Logback은 널리 쓰이는 Logging Framework로 현재 스프링 부트의 기본 프레임 워크이다. Logback은 logback-core, logback-classic, logback-access 세 가지 모듈로 나뉜다.

- logback-core: 다른 두 모듈을 위한 기반 역할을 한다. Appender와 Layout 인터페이스가 이 모듈에 속한다
- logback-classic: logback-core에서 확장된 모듈로 SLF4J를 구현하여, 다른 Logging Framework간 빠르게 전환할 수 있다
- logback-access: Tomcat, Jetty 같은 Servlet Container와 통합되어 HTTP 엑세스 로그 기능을 제공한다. logback-classic, SLF4J와 무관하다. 어플리케이션 레벨이 아닌 컨테이너 레벨에 설치돼야 한다

### Log4j2
Apache Log4j2는 오랫동안 사용된 Logging Framework인 Log4j 1.x 보다 훨씬 향상된 기능을 제공한다. 그리고 현재 널리 쓰이고 있는 Logback에 대한 많은 개선 사항을 제공한다. 가장 최근에 나타난 프레임워크이다.

Log4j API는 로깅 퍼사드이기 때문에 Log4j 구현체와 함께 쓰이거나 Logback과 같은 다른 로깅 구현체의 앞 단에서 사용될 수 다. Log4j는 SLF4J에 비해 몇가지 장점을 가지고 있다.

- 단순 String 뿐 아니라 Message 객체를 지원한다
- 람다 표현을 지원한다
- SLF4J에 비해 많은 로깅 메서드를 지원한다

그리고 Log4j2는 LMAX Disruptor 라이브러리르 기반으로 Asynchronous Logger를 제공해 성능을 크게 향상했다. Asynchronous Logger는 기존의 Log4j 1.x와 Logback에 비해 18배 더 높은 처리량과 낮은 지연시간을 제공한다.


## Bench Mark Test
https://www.loggly.com/blog/benchmarking-java-logging-frameworks/
### File Appender
<img width="375" alt="image" src="https://user-images.githubusercontent.com/60502370/163740600-7e2bff9e-8bc7-474a-a801-f0d5306b56f3.png">

- Synchronous, Asynchronous Appender를 사용할 때 모두 Logback이 Log4j2보다 더 나은 성능을 보지만, 그 차이가 미미하다
- 하지만 Asynchronous Appender를 사용했을 때 Logback의 메시지 소실률이 36%에 달한다

### Syslog Appender
<img width="383" alt="image" src="https://user-images.githubusercontent.com/60502370/163740814-53c6feeb-80ed-4acc-b9ca-b62344211d81.png">

- Log4j2는 TCP에서 로그 메시지의 동기/비동기 전송에서 유실률이 각각 0%, 3%로 크지 않았다
- UDP에서는 동기 전송 했을 때 Log4j2가 Logback보다 조금 빨랐고, 비동기 전송에서 Logback의 메시지 유실률은 61%에 달한다

### Asynchronous loggers
Log4j2는 LMAX의 Disruptor 라이브러리를 기반으로 Asynchronous logger를 제공한다. Asynchronous logger와 Asynchronous Appender의 가장 큰 차이점은 버퍼로 Queue를 사용하는 것이 아니라 LMAX Disruptor를 사용하는 것이다. LMAX Disruptor를 사용하면 Asynchronous Appender를 사용하는 것보다 더 빨리 제어 권한을 Logging Framework에서 어플리케이션으로 반환할 수 있다.

<img width="389" alt="image" src="https://user-images.githubusercontent.com/60502370/163741677-0cf91a0f-a00b-4c7d-9e80-29e6c9bf54e3.png">

- Asynchronous logger는 로그 메시지에서 Location 정보(클래스 이름, 호출 메서드 정보 등)이 제외됐을 때 더 나은 성능을 보인다
- Location 정보를 제거 했을 때는 모든 부분에서 Logback보다 더 나은 성능을 보인다

## 결론
- 대용량 트래픽을 받으면서 로그를 많이 남겨야 하는 경우는 Log4j2와 LMAX Disruptor를 사용하여 Asynchronous Logger를 사용하는 것이 좋겠다
  - 하지만 버퍼를 사용하기 때문에 추가적인 메모리 공간이 필요하고 CPU 사용이 증가할 수 있다는 점을 고려해야 한다
- 이외에는 Logback과 Log4j의 큰 차이점이 없기 때문에 어느것을 사용해도 상관이 없다
  - Spring boot의 기본인 logback을 사용하고, 로깅에서 병목이 발생하면 그 떄 Log4j2로 전환하는 것이 좋겠다

## 참고
- https://stackify.com/compare-java-logging-frameworks/
- https://www.loggly.com/blog/benchmarking-java-logging-frameworks/
- https://gmlwjd9405.github.io/2019/01/04/logging-with-slf4j.html
- https://livenow14.tistory.com/64
- https://logging.apache.org/log4j/2.x/
- https://logback.qos.ch/