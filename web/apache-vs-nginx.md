## 목차
- [Apache VS Nginx](#apache-vs-nginx)
  - [Architecture](#architecture)
    - [Apache - Process Driven](#apache---process-driven)
    - [NGINX - Event Driven](#nginx---event-driven)
  - [Performance](#performance)
    - [Static content](#static-content)
    - [Dynamic content](#dynamic-content)
  - [Configuration](#configuration)
    - [Apache - Distributed](#apache---distributed)
    - [Nginx - Centralized](#nginx---centralized)
  - [Request Interpretation](#request-interpretation)
    - [Apache - Passes File system location](#apache---passes-file-system-location)
    - [Nginx - Passes URI to interpret requests](#nginx---passes-uri-to-interpret-requests)
  - [Flexibility](#flexibility)
    - [Apache](#apache)
    - [Nginx](#nginx)
  - [Apache를 선택하는 경우](#apache를-선택하는-경우)
  - [Nginx를 선택하는 경우](#nginx를-선택하는-경우)
  - [참고](#참고)
- [Apache와 Nginx](#apache와-nginx)
  - [Apache HTTP Server의 등장](#apache-http-server의-등장)
    - [Apache - Process Driven Model](#apache---process-driven-model)
  - [Apache HTTP Server의 한계 - C10K](#apache-http-server의-한계---c10k)
  - [Nginx의 등장](#nginx의-등장)
    - [Nginx가 수 많은 커넥션을 유지하는 방법](#nginx가-수-많은-커넥션을-유지하는-방법)
  - [Nginx의 구조](#nginx의-구조)
    - [Nginx의 동작방식](#nginx의-동작방식)
    - [Nginx의 성장](#nginx의-성장)
  - [Apache HTTP Server의 개선](#apache-http-server의-개선)
  - [Apache와 Nginx의 기능적 비교](#apache와-nginx의-기능적-비교)
    - [Static content](#static-content-1)
    - [Dynamic content](#dynamic-content-1)
    - [Configuration](#configuration-1)
    - [Flexibility](#flexibility-1)
    - [Security](#security)
  - [결론](#결론)

# Apache VS Nginx
## Architecture
### Apache - Process Driven
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/165661348-3483c9ef-8d67-43a9-8601-0656609dd407.png width=700>
</p>

- 아파치는 사용자의 요청당 하나의 프로세스를 할당한다
  - 프로세스를 생성하고 삭제하는 비용이 커 **Prefork** 방식을 채택한다
  - Prefork 모델은 워커 프로세스를 미리 생성해 놓고, 요청에 프로세스를 할당하고 반납하는 방식이다.
  - 트래픽이 급증했고, 웹 페이지 렌더링 시간을 줄이기 위해 TCP 커넥션을 유지하면서 기존 방식은 한계가 있었다
- 새로운 MPM(Multi-Processing Module)의 등장
  - Prefork MPM
    - 기존 방식
  - Worker MPM
    - 적은 수의 자식 프로세스를 생성한다
    - 각각의 자식 프로세스는 다수의 워커 멀티 스레드를 가진다
    - 각각의 워커 스레드는 한 개의 커넥션을 처리한다.
  - Event MPM
    - Worker MPM의 확장된 형태
    - HTTP 요청이 완료된 후 유휴 커넥션을 관리하기 위한 별도의 스레드를 둔다

### NGINX - Event Driven
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/165661807-0e68c9cf-3450-40c0-a92e-872ced202c57.png width=700>
</p>

- 비동기, 이벤트 드리븐 방식으로 커넥션을 처리한다
  - 각 요청만을 전담하는 프로세스나 스레드를 생성하지 않고, 여러 커넥션이나 요청을 하나의 워커 프로세스에서 처리할 수 있는 구조
  - 비동기, 이벤트 드리븐 방식의 가장 큰 장애물은 Bloking. 심지어 Nginx 내부에서도 Blocking 방식의 호출이 일어나고 이를 해결하기 위해 Thread Pool을 사용한다
  - 워커 프로세스가 요청을 처리할 때 잠재적으로 긴 명령이라고 판단하면 작업을 스레드 풀로 넘긴다
  - 스레드 풀의 작업 큐에 작업들이 쌓이고, 스레드들이 작업을 처리한다

## Performance
### Static content
- Apache
  - 파일 기반의 방법으로 정적 컨텐츠를 제공한다
  - css, js 같은 정적 파일을 디스크에서 읽어 온다
- nginx
  - Nginx는 정적 컨텐츠를 제공할 때 PHP를 사용하지 않는다
  - 반면 Apache는 정적 컨텐츠를 제공할 때 Nginx에 비해 오버헤드가 크다

### Dynamic content
- Apache
  - Apache는 동적 컨텐츠를 제공할 때 외부 컴포넌트에 의존하지 않고 독자적으로 할 수 있다
- Nginx
  - Apache 처럼 독자적으로 동적 컨텐츠를 제공할 수 없다
  - 외부 프로세스를 이용해 결과를 전달받아야 한다
  - SCGI, FastCGI와 같은 모듈을 사용해서 동적 컨텐츠를 제공할 수 있다

## Configuration
### Apache - Distributed
- 디렉토리마다 .htaccess 파일을 사용해 추가적인 설정을 할 수 있다
- Apache는 각 경로마다 .htaccess 파일을 확인한다
- 서버, 디렉토리마다 설정을 개별적으로 할 수 있으므로 유연하다
- Apache가 설정 파일을 확인해야 하므로 성능이 느려지고, 보안에 문제가 발생할 수 있다

### Nginx - Centralized
- 메인 서버에 저장된 설정 파일을 외부에서 접근할 수 없다
- 중앙 집중적으로 Nginx를 설정한다
- 웹 서버의 유연한 설정이 불가능 하다
- 디렉토리마다 설정 파일을 확인하지 않으므로 성능상 이점을 가진다

## Request Interpretation
### Apache - Passes File system location
- Apache는 일반적으로 파일 기반의 경로 구조로 요청을 해석한다
- URI로도 가능하다
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/165690433-90c5e642-40c3-44d2-89f7-e2ecc55b2c4c.png width=500>
</p>

### Nginx - Passes URI to interpret requests
- Nginx는 각 구성 요소에 대해 URI를 사용하여 효청을 해석하고 매핑한다
- 요청을 파일 시스템 위치 대신 URI로 전달하면 Nginx가 웹과 프록시 서버로 기능할 수 있게 한다.
- 간단한 설정으로 여러 요청 패턴에 대해서 응답할 수 있다

## Flexibility
### Apache
- 동적 모듈을 통해 웹 서버를 Customization할 수 있음

### Nginx
- 2016년 이전까지 Nginx는 동적 모듈을 제공하지 않았다
- 2016년부터 동적 모듈을 지원한다
- 단, 모듈을 Nginx binary로 컴파일 해야 한다

## Apache를 선택하는 경우
- .htaccess 파일로 서버, 디렉토리마다 설정을 해야하는 경우
- Nginx에서 제공하는 기능에 한계가 있을 경우, Apache는 다양한 모듈로 대체할 수 있다

## Nginx를 선택하는 경우
- 빠른 정적 컨텐츠 처리가 필요한 경우
- 서버에 많은 트래픽이 발생하는 경우

## 참고
- https://themeisle.com/blog/nginx-vs-apache/
- https://serverguy.com/comparison/apache-vs-nginx/
- https://www.nginx.com/blog/nginx-vs-apache-our-view/
- https://www.nginx.com/blog/thread-pools-boost-performance-9x/
- https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/


# Apache와 Nginx
Apache HTTP Server와 Nginx는 현재 전 세계에서 가장 많이 사용되는 웹 서버이다. 두 웹 서버는 정적 콘텐츠를 제공할 뿐 아니라 리버스 프록시 등 다양한 기능을 제공한다. 사실 Nginx는 Apahce의 단점을 개선하기 위한 보조 수단을 목적으로 탄생했었다. 따라서 Apache HTTP Server의 등장과 한계점을 알아보는 것이 두 웹 서버의 차이를 이해하는데 도움이 될 것이다.

## Apache HTTP Server의 등장
Apache HTTP Server의 등장 이전에는 [NCSA HTTPd](https://zetawiki.com/wiki/NCSA_HTTPd)라는 웹 서버가 존재했다. NCSA HTTPd는 세계에서 2번째로 개발된 웹서버로 CGI(Common Gateway Interface)를 처음으로 도입하여 동적 컨텐츠를 제공할 수 있었다. 한때 95% 점유율을 차지했으나 여러가지 버그가 있었고, 몇몇 개발자들이 버그를 수정하고 구조를 변경해 오픈소스 웹 서버인 Apache HTTP Server가 탄생했고 이후 대부분 Apache로 전환되었다.

### Apache - Process Driven Model
Apache는 UNIX 계열 OS가 네트워크 커넥션을 형성하는 모델을 사용했다. 즉, Apache는 HTTP 요청이 들어오면 커넥션을 형성하기 위해 프로세스를 생성한다.

<img width="551" alt="image" src="https://user-images.githubusercontent.com/60502370/166135404-40c1a905-4d08-454f-84d6-fa04b2713c97.png">

각 커넥션마다 프로세스를 생성하는 방식은 개발하기 쉽다는 장점이 있었다. 모듈을 개발하여 웹 서버가 필요로 하는 기능을 손쉽게 확장할 수 있었기 떄문이다.

## Apache HTTP Server의 한계 - C10K
각 커넥션마다 새로운 프로세스를 생성하는 방식은 많은 비용이 발생하였다. 프로세스를 매번 생성하는 비용을 줄이기 위해서 Prefork 방식도 사용했지만, 미리 생성해둔 프로세스 개수보다 커넥션의 수가 많은 경우 새로운 프로세스를 생성해야 했다.

컴퓨터의 보급이 빨라지고 웹이 발전함에 따라 웹 서버에 트래픽이 급속히 증가했다. 어느 날 웹 서버가 1만개 이상의 커넥션을 유지할 수 없는 현상이 발생했고 이러한 문제점을 **C10K 문제** 라 한다.

C10K 문제의 원인은 하드웨어가 아니었다. 원인은 Apache의 구조적 한계 때문이었다. Apache는 각 커넥션마다 프로세스를 생성하고 할당한다. 이로 인해 서버에서 유지해야 할 커넥션의 수가 많아지면서 메모리 부족현상을 야기했고, 컨텍스트 스위칭이 빈번하게 발생해 CPU의 부하가 커졌다.

Apache의 구조는 수 많은 커넥션을 감당하기에 적절하지 않았다. 이러한 Apache의 한계를 극복하기 위해 나타난 것이 Nginx이다

## Nginx의 등장
Nginx는 Apache가 많은 커넥션을 유지하지 못하는 단점을 해결하기 위한 보조 수단으로 탄생했다. Nginx를 통해 Apache의 한계를 극복한 방법은 간단했다. Apache의 앞에 Nginx를 두어 클라이언트의 커넥션을 유지하고 정적 컨텐츠를 제공하고, 동적인 컨텐츠는 Apache가 제공하는 것이 었다.

<img width="551" alt="image" src="https://user-images.githubusercontent.com/60502370/166135919-9a28dd40-8d8a-4289-be39-9e2b1280dd9e.png">

### Nginx가 수 많은 커넥션을 유지하는 방법
Nginx는 Apache가 감당하지 못하는 많은 커넥션을 유지하기 위해 탄생했다. 이를 위해서 Nginx는 비동기, 이벤트 기반의 구조를 채택했다. 이러한 구조는 각 커넥션을 전담하는 프로세스들을 생성하는 것이 아니라 싱글 스레드를 가진 프로세스 하나가 여러 커넥션을 유지하는 것이다.

<img width="551" alt="image" src="https://user-images.githubusercontent.com/60502370/166136141-4709e2ec-367f-406c-b7e6-3fa747fe5d0f.png">

Nginx는 일반적으로 CPU 코어 개수 만큼의 프로세스를 생성한다. 이러한 특징 때문에 서버의 메모리 부족 현상이나 컨텍스트 스위칭으로 인한 CPU의 부하 증가와 같은 현상이 발생하지 않는다.

## Nginx의 구조

<img width="551" alt="image" src="https://user-images.githubusercontent.com/60502370/166136537-6af8fc03-61f4-4050-9054-1a841939e60f.png">

- Master Process: 설정 파일을 읽거나 포트에 바인딩하고 자식 프로세스를 생성하는 등 특권명령을 수행한다
- Cache Loader: 디스크 기반의 캐시를 메모리에 로드한 다음 종료된다. 보수적으로 스케줄링 되므로 자원 소모가 적다
- Cache Manager: 주기적으로 실행되며 디스크 캐시에 저장된 데이터를 제거하여 설정된 캐시 크기 이내로 유지한다
- Worker Process: 네트워크 커넥션 처리, 디스크에 컨텐츠를 읽고 쓰는 등 모든 작업을 수행한다.

### Nginx의 동작방식

쉽게 말하자면, Nginx는 이벤트 핸들러이다. 커널로부터 커넥션에서 발생하는 모든 이벤트들을 전달 받아 처리한다. 

<img width="551" alt="image" src="https://user-images.githubusercontent.com/60502370/166136996-42db1286-7675-4d93-b6e7-c5dbc05fb477.png">

이벤트는 커넥션 생성, 해제, 타임아웃, 에러 등 커넥션에서 발생하는 모든 것들을 의미한다. 커널은 Nginx에게 한 번에 많은 이벤트를 전달한다. Nginx의 Worker 프로세스는 커널로 부터 전달받은 이벤트를 큐에 저장하고 하나씩 처리해 나간다.

<img width="551" alt="image" src="https://user-images.githubusercontent.com/60502370/166137189-b59576c8-c3a3-4a7e-aad8-03dfad3eee20.png">

만약 처리 시간이 긴 명령을 수행하는 경우는 어떨까? 예를 들어 디스크 I/O와 같이 시간이 오래 걸리는 작업의 경우 다음 이벤트를 처리하기 까지 대기 시간이 길어질 것이다. 이러한 문제점을 해결하기 위해서 Nginx는 스레드 풀을 사용한다.

<img width="551" alt="image" src="https://user-images.githubusercontent.com/60502370/166137317-5237800f-6ac1-417a-9d5b-b0cb911accef.png">

Nginx의 스레드 풀은 처리할 작업을 저장하는 태스크 큐와 작업을 처리하기 위한 스레드들로 이루어져 있다. 만약 Worker 프로세스가 잠재적으로 시간이 오래걸리는 작업이라고 판단하면, Worker 프로세스가 직접 작업을 처리하는 것이 아니라 스레드 풀의 태스크 큐에 작업을 전달한다. 그리고 Worker 프로세스는 바로 다음 이벤트를 처리하고, 스레드 풀의 유휴 스레드가 태스크 큐에서 작업을 꺼내 처리한다.

### Nginx의 성장
스마트 폰이 보급되면서 인터넷 환경이 변화했다. 동시 커넥션이 급증했고 페이스북, 트위터, 인스타그램 등 SNS가 성행함에 따라서 다양한 정보를 실시간으로 제공해야 했다. 이러한 환경의 변화는 Nginx가 크게 성장하는 계기가 되었다. 빠른 정적 컨텐츠를 제공할 수 있고 수 많은 커넥션을 감당할 수 있었기 때문이다.

## Apache HTTP Server의 개선
Apache도 구조적 단점을 개선하기 위해서 많은 변화가 이루어지고 있다. Apache의 가장 큰 특징인 모듈화된 설계를 통해 네트워크 커넥션을 형성하고 요청을 처리하기 위한 방식을 개선했는데, 개선된 처리 방식을 다중처리 모듈(Multi-Processing Module, MPM)이라 한다.

**Prefork MPM**은 각 커넥션마다 매번 프로세스를 생성하는 비용을 줄이기위해 미리 프로세스를 생성하고 커넥션에 할당하는 방식이다. 생성된 프로세스의 수보다 많은 커넥션을 유지해야 하는 경우 새로운 프로세스를 생성한다.

**Worker MPM**은 Prefork MPM보다 적은 수의 프로세스를 생성한다. 각 프로세스는 커넥션을 전담하는 멀티 스레드로 이루어져 있다.

**Event MPM**은 Worker MPM의 확장된 버전이다. HTTP 요청을 처리한 후 유휴 커넥션을 유지하기 위한 별도의 스레드를 둔다.

특히 Apache 2.4에서는 캐시, 프록시 모듈, 세션 제어, 비동기 읽기 및 쓰기 지원 기능과 같은 성능 개선을 최우선으로 삼았다.

## Apache와 Nginx의 기능적 비교
이제 Apache와 Nginx의 기능을 비교해본다.

### Static content
- Apache
  - 파일 기반의 방법으로 정적 컨텐츠를 제공한다
- Nginx
  - Apache가 정적 컨텐츠를 제공하기 위해 여러 설정의 주의 깊게 설정해야 하는 반면 Nginx는 Apache에 비해 오버헤드가 크지 않다.
  - Nginx가 Apache에 비해 약 2배 이상 빠르다

### Dynamic content
- Apache
  - Apache는 동적 컨텐츠를 제공할 때 외부 컴포넌트에 의존하지 않고 독자적으로 할 수 있다
- Nginx
  - Apache 처럼 독자적으로 동적 컨텐츠를 제공할 수 없다
  - 외부 프로세스를 이용해 결과를 전달받아야 한다
  - SCGI, FastCGI와 같은 모듈을 사용해서 동적 컨텐츠를 제공할 수 있다

### Configuration
- Apache: Distributed
  - 디렉토리마다 .htaccess 파일을 사용해 추가적인 설정을 할 수 있다
  - Apache는 각 경로마다 .htaccess 파일을 확인한다
  - 서버, 디렉토리마다 설정을 개별적으로 할 수 있으므로 유연하다
  - Apache가 설정 파일을 확인해야 하므로 성능이 느려지고, 보안에 문제가 발생할 수 있다
- Nginx: Centralized
  - 메인 서버에 저장된 설정 파일을 외부에서 접근할 수 없다
  - 중앙 집중적으로 Nginx를 설정한다
  - 웹 서버의 유연한 설정이 불가능 하다
  - 디렉토리마다 설정 파일을 확인하지 않으므로 성능상 이점을 가진다

### Flexibility
- Apache
  - 동적 모듈을 통해 웹 서버를 Customization할 수 있다
  - 60가지 이상의 공식 모듈을 제공한다
- Nginx
  - 2016년부터 동적 모듈을 지원한다
  - Apache에 비해 모듈 개발이 어렵고 다양한 모듈이 없다

### Security 
- Apache
  - Apache 2.4.x 기준 Security Impact Level moderate, important, critical의 각 빈도수([참고](https://httpd.apache.org/security/vulnerabilities_24.html))
  - moderate: 26
  - important: 22
  - critical: 2
- Nginx
  - Nginx Open Source Security Advisories severity level medium, major의 각 빈도수([참고](https://nginx.org/en/security_advisories.html))
  - medium: 14
  - major: 11

## 결론
Apache HTTP Server는 기본적으로 각 커넥션을 전담하는 프로세스를 생성한다. 이러한 특징으로 모듈을 쉽게 확장하여 상황에 맞게 웹 서버를 커스터 마이징할 수 있다는 장점이있다. 하지만 프로세스를 많이 생성하기 때문에 메모리를 많이 차지하고, 컨텍스트 스위칭이 발생하서 CPU의 부하가 커질 수 있다는 단점이 있다. Apache는 이러한 단점을 해결하기 위해 Worker MPM이나 Event MPM 등 멀티 스레드를 활용한 방식을 내놓거나, 비동기 읽기/쓰기 등 성능 향상에 초점을 두고 계속해서 발전해 나가고 있다.

Nginx는 태생부터 수 많은 커넥션을 유지하고 정적 컨텐츠를 빠르게 제공할 수 있도록 비동기, 이벤트 기반 구조를 채택하여 설계되었다. 서버의 코어 개수만큼의 Worker 프로세스를 생성하고, 이 Worker 프로세스가 이벤트 큐에 담긴 이벤트들을 처리한다. 그리고 처리 시간이 긴 작업의 경우 스레드 풀을 활용하여 Non-blocking으로 처리하고 있다. 

현재 상황에서 어느 하나가 무조건 뛰어나다라고 단언할 수 없다. 두 웹 서버가 제공하는 기능 측면에서 차의는 거의 없다고 생각한다. 만약 디렉토리나 서버마다 서로 다른 설정이 필요하고 기능의 확장이 필요한 경우 Apache를 선택할 수 있겠다. 만약 수 많은 트래픽을 감당해야 하고 정적 컨텐츠를 빠르게 제공해야하며 메모리, CPU 자원이 부족하다면 Nginx를 선택할 수 있다.