# ARP

단말간 통신을 위해서는 MAC 주소와 IP 주소를 알아내야한다. IP 주소는 DNS를 사용해 알아낼 수 있다. 이처럼 상대방의 MAC 주소를 알아내기 위한 프로토콜이 ARP이다.

## ARP란?
ARP는 3계층의 논리적 주소와 2계층의 물리적 주소 사이에 관계가 없는 프로토콜에서 두 주소를 서로 연결하는 역할을 한다. 통신을 시도할 때 IP 주소와 Port는 미리 알고있지만 상대방의 MAC 주소는 알 수 없다. 이떄 ARP 브로드캐스트를 이용해 전체 네트워크에서 상대방의 MAC 주소를 질의 해야한다.

### ARP 테이블
패킷을 보낼 때마다 ARP 브로드캐스트를 하면 네트워크 통신 효율이 저하된다. 이 때문에 ARP 테이블을 구성하여 메모리에 이를 저장해두고 재사용한다. ARP 테이블을 오래 유지하는 것이 성능에 좋지만 논리적인 주소는 변경될 수 있기 때문에 일정 시간이 지나면 ARP 테이블에서 삭제한다.

### ARP 동작 방식
ARP 패킷은 여러가지 필드 중 ARP 데이터에 사용되는 송신자 하드웨어 MAC 주소, 송신자 IP 프로토콜 주소, 대상자 MAC 주소, 대상자 IP 프로토콜 주소를 중요하게 생각한다.

1. 송신자가 수신자에게 데이터를 보낼때 수신자의 MAC 주소를 알아내기 위해 ARP 요청을 브로드 캐스트한다
   - 도작치를 FF-FF-FF-FF-FF-FF로 하여 브로드캐스트한다.
   - 대상자 MAC 주소는 00-00-00-00-00-00으로 한다
2. 같은 네트워크의 모든 호스트에게 요청이 전달된다. 단말은 ARP 프로토콜 필드의 대상자 IP가 자신과 일치하는 지 확인하고 폐기하거나 응답을 보낸다
3. 송신자는 이제 대상자의 MAC 주소를 채워 응답한다

### GARP
Gratuitous ARP는 대상자 IP 필드에 자신의 IP주소를 채워 ARP 요청을 보낸다. 이는 자신의 IP와 MAC 주소를 알리기 위함이다. GARP를 사용하는 이유는 다음과 같다.
1. IP 주소 충돌 감지
2. 상대방의 ARP 테이블 갱신