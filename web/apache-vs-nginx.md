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