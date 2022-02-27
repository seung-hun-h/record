# Message Queue
메시지 큐(Message Queue)는 프로세스 또는 프로그램 간 데이터를 교환할 때 사용하는 통신 방법 중 하나로, 메시지 지향 미들웨어(Message Oriented Middleware)를 구현한 시스템을 의미한다.

## MOM: Message Oriented Middleware
독립적인 정보 단위인 메시지를 운반하는 통신 채널을 이용해서 애플리케이션간 데이터를 교환하는 시스템 구조를 의미한다. MOM 기반의 시스템에서는 메시지를 비동기적으로 주고 받는다.

MOM에서 메시지를 비동기적으로 주고 받기 위해서는 중간에 메시지를 보관할 소프트웨어가 필요하다. 이 소프트웨어를 메시지 큐(Message Queue)라고 한다. 이 메시지 큐의 존재로 인해 MOM은 요청-응답 구조를 위반한다고 볼 수 있다.

<img width="347" alt="image" src="https://user-images.githubusercontent.com/60502370/155870244-b6b5e84e-cfad-47dc-9506-e2efa35d1376.png">


### MOM의 기능
- 복잡한 시스템에서 메시지의 분배가 가능해진다
- 서로 다른 두 애플리케이션이나 플랫폼을 연결 할 수 있다
- 다른 IT 조직간 메시지 전달을 구현할 수 있다

## Message Queue
메시지 큐는 메시지의 무손실을 보장하는 비동기 통신을 지원하는 컴포넌트이다. 메시지 버퍼의 역할을 하며 비동기적으로 전송한다. 생산자 또는 발행자가 메시지를 만들어 메시지 큐에 발행하고, 소비자 혹은 구독자가 메시지를 받아 그에 맞는 동작을 수행한다.

<img width="363" alt="image" src="https://user-images.githubusercontent.com/60502370/155870432-ac6e017e-c016-459c-998f-e727e3053551.png">

### 메시지 큐의 장점
- Asynchronous
- Loose Coupling
- Scalable
- Resilience
- Guarantees

메시지 큐를 이용하면 서비스 또는 서버 간 결합이 스즌해져 규모의 확장성이 보장되어야 하는 안정적 애플리케이션을 구성하기 좋다. 그리고 비동기 통신을 지원해 생산자는 소비즈 프로세스가 다운되어 있어도 메시지를 발행할 수 있고, 소비자는 생산자 서비스가 가용한 상태가 아니더라도 메시지를 수신할 수 있다.

### 메시지 큐 사용예시
메시지 큐는 비동기적 특성을 가지고 있기 때문에 처리 실패시 치명적인 영향을 미치는 핵심적인 작업 보다는 부가적인 기능에 사용하는 것이 적합하다.

- 이메일 전송
  - 비밀번호 재설정이나 회원 가입 인증을 위한 이메일 전송 서비스를 메시지 큐에 넣어 비동기처리
- 블로그 포스팅
  - 블로그 포스트에 고용량 이미지가 포함된 경우
  - 이미지가 저장소에 전송되고, 업로드된 이미지의 최적화 작업이 메시지 큐에 담긴다
  - 최적화 작업을 비동기적으로 처리한다
- 사진 보정
  - 사진 보정은 시간이 오래 걸릴 수 있는 작업이므로 Worker Process가 작업을 메시지 큐에서 꺼내어 비동기적으로 작업할 수 있다.

## 참고
- https://tecoble.techcourse.co.kr/post/2021-09-19-message-queue/
- https://www.geeksforgeeks.org/what-is-message-oriented-middleware-mom/
- [가상 면접 사례로 배우는 대규모 시스템 설계 기초](http://www.yes24.com/Product/Goods/102819435)

