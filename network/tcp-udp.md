# TCP와 UDP

2, 3 계층은 주소를 사용하여 목적지를 정확하게 찾아가는 것이 목표였다. 4계층에서 동작하는 프로토콜은 목적지 단말 안에서 동작하는 여러 애플리케이션 중 통신해야 할 **목적지 프로세스**를 정확히 찾아가고 패킷 순서가 바뀌지 않도록 잘 조합해서 원래 데이터를 만들어내기 위한 역할을 한다.

4계층의 프로토콜인 TCP와 UDP는 포트 번호를 통해 출발지와 목적지를 표현한다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/140883809-c054cc33-04ad-465c-9477-e250acaa555b.png width=700>
</p>

## TCP
TCP는 신뢰할 수 없는 공용망에서 정보유실 없는 통신을 보장하기 위해 세션을 안전하게 연결하고 데이터를 분할하고 패킷이 잘 전송되었는지 확인하는 기능이있다.

패킷에 번호(Sequence Number)를 부여하고 잘 전송되었는지에 대해 응답(Ackowledge Number)한다. 얼마나 보내야 수신자가 잘 처리할 수 있는지 전송 크기(Window Size)까지 고려해 전송한다.

### 패킷 순서, 응답 번호
TCP는 분할된 패킷을 수신측이 잘 조합하도록 패킷에 순서를 주고 응답 번호를 부여한다. 패킷의 순서를 시퀀스 번호, 응답 번호를 부여하는 것을 ACK라고한다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/140884669-6524e2de-e569-4e73-afdd-67cfa68e2954.png width=500>
</p>

1. 출발지에서 시퀀스 번호를 0으로 보낸다
2. 수신 측에서 0번 패킷을 잘 받았다는 의미로 응답 번호(ACK)에 1을 적어 응답한다.
   - 수신 측에서는 자신이 처음 보내는 패킷이므로 자신 패킷 시퀀스 번호는 0번을 부여한다.
3. 송신 측은 시퀀스 번호를 1로, ACK는 0번 패킷을 잘 받았다는 의미로 1로 부여해 전송한다.

### 윈도 사이즈와 슬라이딩 윈도
송수산자간 패킷을 주고 받을 때 거리가 멀리 떨어져있으면 왕복 지연시간이 늘어나므로 응답을 기다리는 시간이 지연된다.

따라서 한 번에 많은 패킷을 보내고 한꺼번에 응답을 받아야한다. 한 번에 받을 수 있는 데이터의 크기를 윈도 사이즈라고 하고 네트워크 상황에 따라 윈도 사이즈를 조절하는 것을 슬라이딩 윈도라고 한다.

### 오류제어
ARQ(Automatic Repeat Request) 기법을 사용해 프레임이 손상되었거나 손실되었을 경우 재전송을 통해 오류를 복구한다.

- Stop and Wait ARQ: 송신측에서 패킷을 전송하고 NAK를 받을 경우 재전송하는 기법이다.
  - 데이터나 ACK를 유실했을 경우 타임 아웃이 발생해 재전송된다.

- Go-Back-n ARQ: 전송된 프레임이 손실되거나 유실되어 타임 아웃이 발생한 경우 마지막으로 확인된 패킷 이후의 전체 패킷을 재전송한다
- Selective-Reject ARQ: 손실되거나 유실된 프레임만 재전송하는 기법이다. 프레임의 재정렬을 수행해야하므로 별도의 버퍼가 필요하다.
- 
### 흐름제어
송신측과 수신측 사이의 데이터 처리 속도 차이를 제어하기 위한 기법이다.

- 정지-대기: 매번 전송한 패킷에 대한 응답을 받아야 다음 패킷을 전송할 수 있다
  
- 슬리이딩 윈도우: 수신측에서 설정한 윈도우 크기만큼 확인 응답 없이 한 번에 여러 패킷을 전송할 수 있다.

### 혼잡 제어
- **Addictive Increase**

전송한 패킷에 대해서 ACK를 받은 경우 윈도우 사이즈를 1씩 증가시키는 방법

- **Multiplicative Decrease**


ACK를 받지 못한 경우 윈도우 사이즈를 절반으로 줄이는 방법

- **Slow start**


윈도우 사이즈를 1로 시작한다. 전송에 성공한 패킷의 수 만큼 윈도우 사이즈를 증가시키는 방법이다. 즉 윈도우 크기가 1, 2, 4, 8 , 16, ... 2의 지수승 만큼증가한다.

그리고 네트워크 혼잡이 감지된 경우 그때 윈도우 사이즈를 Threshold로 기억하고 윈도우 사이즈를 1로 줄인다. 다시 윈도우를 2의 지수만큼 증가하다가 Threshold의 절반 크기가 된 경우 윈도우 사이즈를 1씩 늘린다.

- **Fast Retransmit**

패킷 하나가 중간에 유실된 경우 수신측은 유실된 패킷의 다음 시퀀스를 ACK에 실어 보낸다. 수신측은 유실된 패킷의 시퀀스를 중복적으로 보내고 시퀀스가 3번이 중복되면 송신측은 해당 패킷을 재전송한다.

- **Fast Recovery**

네트워크 혼잡이 감지되면 윈도우 사이즈를 1이 아닌 절반으로 줄이고 선형증가하는 방법이다. 이 정책은 처음 혼잡 상황을 겪은 뒤에는 계속 선형증가한다.


### 3way-handshake

TCP는 유실없는 안전한 통신을 위해 사전 연결 작업을 수행한다. 패킷 네트워크에서는 동시에 많은 상대방과 통신하므로 정확한 통신을 위해서 통신 전, 각 통신에 필요한 리소스를 미리 확보하는 것이 중요하다. 3번의 패킷 교환을 통해 통신을 준비하여 3way handshake라 한다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/140899565-9a7169dd-74db-4219-a656-996bd5529cdd.png width=500>
</p>

1. 송신자가 SYN 필드를 1로 표기해 패킷을 전송한다
   - 이때 자신이 사용할 패킷의 첫 시퀀스 번호를 적어보낸다.
   - 포트 번호는 유한 범위내에서 설정되므로 시퀀스 번호를 순차적으로 설정할 경우 이전 통신의 패킷으로 오해할 수 있기 때문이다.


2. 수신자는 ACK와 SYN 필드를 1로 표기해 응답한다
   - 자신이 보내는 첫 패킷이므로 SYN을 함께 보낸다
   - 연결 신호를 허락하는 의미로 전달한다.

3. 송신자가 ACK를 1로 표기해 전달한다.

### 4way-handshake
TCP는 세션을 안전하게 종료하기 위해 4개의 패킷을 교환한다. 이를 4way-handshake라 한다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/149465776-7bdece24-58b2-4fdc-9cc4-25810c9d8999.png width=500>
</p>

1. 송산자가 연결을 종료하기 위해 FIN에 1을 부여하여 패킷을 전송한다
2. 수신자는 요청 확인의 의미로 ACK에 1을 표기하여 패킷을 전송한다.
3. 수신자는 연결 종료 준비를 완료한 후 FIN에 1을 표기하여 송신자에게 패킷을 전송한다
4. 송신자는 연결 종료 확인의 의미로 ACK에 1을 표기하여 패킷을 전송한다
5. 송신자는 아직 도착하지 못한 패킷이 있을 경우를 대비해 기다렸다가 연결을 종료한다.

## UDP
UDP는 4계층에서 가져야할 특징을 거의 가지고 있지 않다. UDP는 신뢰성있는 데이터의 전송을 보장하지 않는다. 손상되거나 유실된 데이터에 대한 조치를 하지 않으므로 헤더가 가볍다.

UDP는 실시간 스트리밍과 같이 시간에 민감한 애플리케이션을 사용하거나 사내 방송과 같은 단방향으로 다수의 단말과 통신해 응답을 받기 어려운 환경에서 주로 사용된다. 보통 음성이나 동영상과 같은 데이터를 전송할 때 사용된다. 30 프레임의 동영상에 1 프레임이 빠져도 크게 이상한점을 느끼지 못하기 때문이다.

UDP는 TCP와 달리 연결을 사전에 확립하는 작업이 없다. 대신 UDP에서 첫 데이터는 리소스 확보를 위해 인터럽트를 거는 용도로 사용한다. 따라서 연결 확립은 TCP로 하고 애플리케이션끼리 모든 준비를 마친 후 실제 데이터만 UDP를 사용한다.

같은 동영상 스트리밍이더라도 넷플릭스처럼 시간에 민감하지 않는 단일 시청자를 위한 연결은 TCP를 사용한다. 원활한 시청을 위해 미리 데이터를 받아 놓고 네트워크에 문제가 생기더라도 동영상이 끊기지 않도록 캐시에 저장한다. 실시간 화상 회의와 같은 시간에 민감한 데이터에서는 UDP를 사용한다.
