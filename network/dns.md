# DNS

## DNS란?
DNS는 도메인 주소를 IP 주소로 변환하는 역할을 한다. 

<img width="465" alt="image" src="https://user-images.githubusercontent.com/60502370/155866541-a90e825b-356b-485b-9d69-729850302e9c.png">

### DNS의 필요성
- 사용자들이 IP 주소 보다 도메인 주소에 익숙하고, 도메인 주소가 기억하기 쉽다
- 서버 IP 변경에 쉽게 대처할 수 있다
- 최근 MSA 기반의 서비스 설계가 많아지면서 다수의 API를 이용해 서비스간 API 호출이나 인터페이스가 많아졌다
  - IP로 서비스간 연결을 구현하면 어느 한 서비스의 IP 변경이 필요한 경우 여러가지 설정을 변경하거나 프로그램을 재배포 해야한다.
  - 도메인 주소를 사용하면 DNS 서버에서 간단한 설정 변경만 하면된다

## DNS 구조와 명명 규칙

<img width="431" alt="image" src="https://user-images.githubusercontent.com/60502370/155866600-b4260903-7aa0-4dd5-a0c4-f52f50cd9a3c.png">

맨 뒤에 생략된 루트(.)을 시작으로 Top-level인 com에서  Third-level인 www와 같이 뒤에서 앞으로 해석한다.

- 도메인 계층은 최대 128 계층까지 구성할 수 있다
- 계층별 길이는 최대 63바이트를 사용할 수 있다
- 전체 도메인 이름의 길이는 최대 255 바이트까지 이다.
- 알파벳, 숫자, - 만 사용할 수 있고, 대소문자 구분이 없다

DNS 서버는 사용자가 쿼리한 도메인에 대한 값을 직접 갖고 있거나 캐시에 저장된 정보를 이용해 응답한다. 만약 DNS 서버가 정보를 가지고 있지 않는 경우 루트 DNS 서버에 질의한다.

### 루트 도메인
- 도메인을 구성하는 최상위 영역
- 루트 DNS는 전 세계에 13개가 존재한다
- DNS 서버를 설치하면 루트 DNS의 IP 주소를 기록한 힌트 파일을 가지고 있다

## DNS 동작 방식

<img width="541" alt="image" src="https://user-images.githubusercontent.com/60502370/155866772-c954bb21-baab-4631-bce8-ed35c7c62407.png">

1. 로컬의 hosts 파일을 확인한다
2. 로컬에 존재하지 않는 경우 DNS 서버에 IP 주소 변환을 요청한다
   1. DNS 서버는 DNS 캐시를 확인한다
3. DNS 캐시에도 존재하지 않으면 루트 DNS 서버에 질의한다
4. 루트 DNS 서버에 존재하지 않으면 .net 도메인을 관리하는 서버의 IP 주소를 전달한다
5. .net 도메인을 관리하는 TLD DNS 서버에 질의한다
6. TLD DNS 서버에 존재하지 않으면 zigispace 도메인을 관리하는 DNS 서버의 IP 주소를 전달한다
7. zigispace를 관리하는 도메인 서버에 질의한다
8. zigispace를 관리하는 도메인 서버는 IP www.zigispace.net의 IP 주소를 전달한다
9. 호스트에 IP 주소를 전달한다