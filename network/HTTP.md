# 1. HTTP

## 1.1 HTTP 개요

HTTP(HyperText Transfer Protocol)은 HTML과 같은 문서를 웹 상에서 클라이언트와 서버간 요청/응답(Request/Response)으로 주고 받을 수 있는 프로토콜이다.

- 주로 HTML 문서를 주고 받는데 사용한다.
- TCP와 UDP를 사용하며, 80번 포트를 사용한다.
- 비연결(Connectionless) 프로토콜이다.
- 무상태(Stateless)프로토콜이다.

**비연결(Connectionless)**
클라이언트가 요청을 서버에 보내고 서버가 적절한 응답을 클라이언트에 보내면 바로 연결이 끊어진다.

**무상태(Stateless)**
연결을 끊는 순간 클라이언트와 서버의 통신은 끝나며 상태 정보를 유지하지 않는다.

**HyperText**
참조를 통해 사용자가 한 문서에서 다른 문서로 즉시 접근할 수 있는 텍스트

## 1.2 HTTP 문제점

- HTTP는 평문 통신이기 때문에 도청이 가능하다.
- 통신 상태를 확인하지 않기 때문에 위장이 가능하다.
- 완전성을 증명할 수 없기 때문에 변조가 가능하다.

# 2. HTTPS

## 2.1 HTTPS 개요

HTTP의 보안 문제를 해결하기위해 대두되었다. HTTPS는 인터넷 상에서 정보를 암호화하는 SSL(Secure Socket Layer) 프로토콜을 이용하여 클라이언트와 서버가 데이터를 주고 받는 프로토콜이다.

- HTTPS의 기본 TCP/IP 포트로 443번 포트를 사용한다.
- HTTPS는 소켓 통신에서 일반 텍스트를 이용하는 대신에, 웹 상에서 정보를 암호화하는 SSL이나 TLS(Transfer Layer Security) 프로토콜을 통해 세션 데이터를 암호화한다.
- 데이터의 적절한 보호를 제공한다. 보호의 수준은 웹 브라우저에서의 구현 정확도와 서버 소프트웨어, 지원하는 암호화 알고리즘에 달려있다.
- HTTPS의 SSL에서는 대칭키 암호화 방식과 공개키 암호화 방식을 모두 사용한다.

TSL 프로토콜은 SSL 프로토콜에서 발전한 것이다.
두 프로토콜의 주요 목표는 기밀성(사생활 보호), 데이터 무결성, ID 및 디지털 인증서를 사용한 인증을 제공하는 것이다.

## 2.2 HTTPS의 필요성

클라이언트가 서버에 HTTP를 통해 웹 페이지나 이미지 정보를 요청하면 서버는 이 요청에 응답하여 요구하는 정보를 제공한다.

웹 페이지는 텍스트이고, HTTP를 통해 이러한 정보를 교환하는 것이다.
이떄 중요한 정보를 중간에서 볼 수 없도록 암호화하는 방법인 HTTPS를 사용하여 보안상 문제를 해결할 수 있다.

## 2.3 HTTPS의 원리

### 2.3.1 공개키 암호화 방식

HTTPS의 핵심적인 암호화 원리는 **공개키 암호화 방식**이다.

- 두 개의 다른 키를 사용한다.
공개키: 모든 사람이 접근 가능한 키
개인키: 각 사용자 자신만이 소유하는 키, 클라이언트-서버 구조에서는 서버가 가지고 있다.
- 두 키 중 하나는 암호에 다른 하나는 복호에 사용된다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c6c28cd5-b97c-44ca-97cb-bbeaaf1d3434/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c6c28cd5-b97c-44ca-97cb-bbeaaf1d3434/Untitled.png)

1. B의 공개키와 개인키 생성
2. B의 공개키는 공개하고 개인키는 개인이 소유
3. A는 B의 공개키로 메시지를 암호화
4. B는 자신의 개인키로 메시지 복호화

### 2.3.2 SSL 동작 방식

SSL 프로토콜은 Netscape 사에서 웹 서버와 브라우저 사이의 보안을 위해 만들어졌다. CA(Certificate Authority)라 불리는 서드 파티로부터 서버와 클라이언트 인증을 하는데 사용된다.

**애플리케이션 서버를 운영하는 기업은 CA를 통해 인증서를 만든다.**

1. 애플리케이션 서버를 운영하는 기업은 HTTPS 적용을 위해 공개키와 개인키를 만든다.
2. 신뢰할 수 있는 CA 기업을 선택하고 인증서 생성을 요청한다.
3. CA는 서버의 공개키, 암호화 방법 등의 정보를 담은 인증서를 만들고 해당 CA의 개인키로 암호화하여 서버에 제공한다.
4. 클라이언트가 SSL로 암호화된 페이지를 요청시 서버는 인증서를 전송한다.

**클라이언트와 서버의 통신 흐름 과정**

1. 클라이언트가 SSL로 암호화된 페이지를 요청한다.
2. 서버는 클라이언트에게 인증서를 전송한다.
3. 클라이언트는 인증서가 신용 있는 CA로부터 서명된 것인지 판단한다. 브라우저는 CA리스트와 해당 CA의 공개키를 가지고 있다. 공개키를 활용하여 인증서가 복호화가 가능하다면 접속한 사이트가 CA에 의해 검토되었다는 것을 의미한다. 따라서 서버가 신용이 있다고 판단한다. 공개키가 데이터를 제공한 사람의 신원을 보장해주는 것으로 이러한 것을 **전자 서명**이라 한다.
4. 클라이언트는 CA의 공개키를 이용해 인증서를 복호화하고 서버의 공개키를 획득한다.
5. 클라이언트는 서버의 공개키를 사용해 랜덤 대칭 암호화키, 데이터 등을 암호화하여 서버로 전송한다.
6. 서버는 자신의 개인키를 이용해 복호화하고 랜덤 대칭 암호화키, 데이터 등을 획득한다.
7. 서버는 랜덤 대칭 암호화키로 클라이언트 요청에 대한 응답을 암호화하여 전송한다.
8. 클라이언트는 랜덤 대칭 암호화키를 이용해 복호화하고 데이터를 이용한다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7503858b-101d-42c6-9b6c-f8c2b17b9e92/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7503858b-101d-42c6-9b6c-f8c2b17b9e92/Untitled.png)

**인증서에 포함된 내용**

- 서버측 공개키
- 공개키 암호화 방법
- 인증서를 사용한 웹서버의 URL
- 인증서를 발행한 기관 이름

## 2.4 HTTPS 장단점

### 2.4.1 장점

- 네트워크 상에서 열람, 수정이 불가능하므로 안전하다.

### 2.4.2 단점

- 암호화를 하는 과정이 웹 서버에 부하를 준다.
- HTTPS는 설치 및 인증서를 유지하는데 추가 비용이 발생한다.
- HTTP에 비해 느리다.
- 인터넷의 연결이 끊긴 경우 재인증 시간이 소요된다.

HTTP는 비연결형으로 웹 페이지를 보는 중 인터넷 연결이 끊겼다가 다시 연결되어도 페이지를 계속 볼 수 있다.

하지만 HTTPS의 경우에는 소켓 자체에서 인증을 하기 때문에 인터넷 연결이 끊기면 소켓도 끊어져 다시 HTTPS 인증이 필요하다.

# 3. HTTP 요청/응답 헤더

## 3.1 일반 헤더(General Header)

요청 및 응답 메시지 모두에서 사용 가능한 일반 목적의 헤더 항목을 의미한다.

- Date: HTTP 메시지를 생성한 일시
`Date: Sat, 2 Oct 2018 02:00:12 GMT`
- Connection: 클라이언트와 서버간 연결에 대한 옵션 설정
`Connection: close`  = 현재 HTTP 메시지 직후에 TCP 접속을 끊는다는 것을 알림
`Connection: Keep-Alive` = 현재 TCP 커넥션을 유지
- Cache-Control
- Pragma
- Trailer

## 3.2 엔터티/개체 헤더(Entity Header)

요청 및 응답 메시지 모두에서 사용가능한 Entity(콘텐츠, 본문, 리소스 등)에 대한 설명 

- Content-Type: 해당 개체에 포함되는 미디어 타입 정보
    - 컨텐츠의 타입(MIME 미디어 타입) 및 문자 인코딩 방식 등을 지정
    - 타입 및 서브타입(type/subtype)으로 구성
    - `Content-Type: text/html; charset-latin-1`
- Content-Language: 해당 개체와 가장 잘 어울리는 사용자 언어
- Content-Encoding: 해당 개체의 데이터 압축 방식
    - `Content-Encoding: gzip, deflate`
    - 만일 압축이 시행되었다면, Content-Encoding, Content-Length 2개 항목을 토대로 압축 해제가능
- **Content-Length**: 전달되는 해당 개체의 바이트 길이 또는 크기(10진수)
    - 응답 메시지 Body의 길이를 지정하거나, 특정 지정된 개체의 길이를 지정함
- **Content-Location**: 해당 개체가 실제 어디에 위치하는가를 알려줌
- **Content-Disposition**: 응답 Body를 브라우저가 어떻게 표시해야 할지 알려주는 헤더
    - inline인 경우 웹페이지 화면에 표시되고, attachment인 경우 다운로드
    - `Content-Disposition: inline`
    - `Content-Disposition: attachment; filename='filename.csv'`
    - 다운로드되길 원하는 파일은 attachment로 값을 설정하고, filename 옵션으로 파일명까지 지정해줄 수 있다.
    - 파일용 서버인 경우 이 태그를 자주 사용
- **Content-Security-Policy**: 다른 외부 파일들을 불러오는 경우, 차단할 소스와 불러올 소스를 명시
    - *XSS 공격*에 대한 방어 가능 (허용한 외부 소스만 지정 가능)
    - `Content-Security-Policy: default-src https:` => https를 통해서만 파일을 가져옴
    - `Content-Security-Policy: default-src 'self'` => 자신의 도메인의 파일들만 가져옴
    - `Content-Security-Policy: default-src 'none'` => 파일을 가져올 수 없음
- **Location**: 리소스가 리다이렉트(redirect)된 때에 이동된 주소, 또는 새로 생성된 리소스 주소
    - 300번대 응답이나 201 Created 응답일 때 어느 페이지로 이동할지를 알려주는 헤더
    - 새로 생성된 경우에 HTTP 상태 코드 `201 Created`가 반환됨
    - `HTTP/1.1 302 Found Location: /`
        - 이런 응답이 왔다면 브라우저는 / 주소로 redirect한다.
- **Last-Modified**: 리소스를 마지막으로 갱신한 일시

## 3.3 요청 헤더(Request Header)

- 요청 헤더는 HTTP 요청 메시지 내에서만 나타나며 가장 방대하다.
- 주요 항목들
    - **Host**: 요청하는 호스트에 대한 호스트명 및 포트번호 (***필수***)
        - Host 필드에 도메인명 및 호스트명 모두를 포함한 전체 URI(FQDN) 지정 필요
        - 이에 따라 동일 IP 주소를 갖는 단일 서버에 여러 사이트가 구축 가능
    - **User-Agent**: 클라이언트 소프트웨어(브라우저, OS) 명칭 및 버전 정보
    - **From**: 클라이언트 사용자 메일 주소
        - 주로 검색엔진 웹 로봇의 연락처 메일 주소를 나타냄
        - 때로는, 이 연락처 메일 주소를 User-Agent 항목에 두는 경우도 있음
    - **Cookie**: 서버에 의해 Set-Cookie로 클라이언트에게 설정된 쿠키 정보
    - **Referer**: 바로 직전에 머물었던 웹 링크 주소
    - **If-Modified-Since**: 제시한 일시 이후로만 변경된 리소스를 취득 요청
    - **Authorization**: 인증 토큰(JWT/Bearer 토큰)을 서버로 보낼 때 사용하는 헤더
        - 토큰의 종류(Basic, Bearer 등) + 실제 토큰 문자를 전송
    - **Origin**
        - 서버로 POST 요청을 보낼 때, 요청이 어느 주소에서 시작되었는지 나타냄
        - 여기서 요청을 보낸 주소와 받는 주소가 다르면 *CORS 에러*가 발생
        - 응답 헤더의 **Access-Control-Allow-Origin**와 관련
- 다음 4개는 주로 HTTP 메세지 Body의 속성 또는 내용 협상용 항목들
    - **Accept**: 클라이언트 자신이 원하는 미디어 타입 및 우선순위를 알림
        - `Accept: */*` => 어떤 미디어 타입도 가능
        - `Accept: image/*` => 모든 이미지 유형
    - **Accept-Charset**: 클라이언트 자신이 원하는 문자 집합
    - **Accept-Encoding**: 클라이언트 자신이 원하는 문자 인코딩 방식
    - **Accept-Language**: 클라이언트 자신이 원하는 가능한 언어
    - 각각이 HTTP Entity Header 항목 중에 `Content-Type, Content-Type charset-xxx, Content-Encoding, Content-Language`과 일대일로 대응됨

### User-Agent
요청 보내는 클라이언트의 정보를 포함하고 있는 요청 헤더이다. 클라이언트 소프트웨어 명칭 밎 버전 정보등이 포함되는데, 이는 서버측에서 상호 운용 문제에 대한 범위를 식별하고 특정 User Agent의 제한을 피하기 위해 응답을 조정하고 브라우저나 운영 시스템에 대한 분석을 하기 위함이다.

 - User-Agent = product *(RWS (product / comment))
 - User-Agent fieldS
   - 1개 이상의 Product identifier
   - 0개 이상의 Product에 대한 Comment
   - Product identfier는 관례상 중요도에 따른 내림차순으로 정렬된다
   - product = token ["/" product-version]

클라이언트는 클라이언트 소프트웨어를 식별하기 위한 필수적인 정보만 제공해야한다. 그리고 동일한 소프트웨어의 연속적인 버전 정보를 표기해선 안된다.


## 3.4 응답 헤더(Response Header)

- 특정 유형의 HTTP 요청이나 특정 HTTP 헤더를 수신했을 때, 이에 응답한다.
- 주요 항목들
    - **Server**: 서버 소프트웨어 정보
    - **Accept-Range**
    - **Set-Cookie**: 서버측에서 클라이언트에게 세션 쿠키 정보를 설정 (RFC 2965에서 규정)
    - **Expires**: 리소스가 지정된 일시까지 캐시로써 유효함
    - **Age**: 캐시 응답. max-age 시간 내에서 얼마나 흘렀는지 알려줌(초 단위)
    - **ETag**: HTTP 컨텐츠가 바뀌었는지를 검사할 수 있는 태그
    - **Proxy-authenticate**
    - **Allow**: 해당 엔터티에 대해 서버 측에서 지원 가능한 HTTP 메소드의 리스트를 나타냄
        - 때론, HTTP 요청 메세지의 HTTP 메소드 OPTIONS에 대한 응답용 항목
            - OPTIONS: 웹서버측 제공 HTTP 메소드에 대한 질의
        - `Allow: GET,HEAD` => 웹 서버측이 제공 가능한 HTTP 메서드는 GET,HEAD 뿐임을 알림 (405 Method Not Allowed 에러와 함께)
    - **Access-Control-Allow-Origin**: 요청을 보내는 프론트 주소와 받는 백엔드 주소가 다르면 *CORS 에러*가 발생
        - 서버에서 이 헤더에 프론트 주소를 적어주어야 에러가 나지 않는다.
        - `Access-Control-Allow-Origin: www.zerocho.com`
            - 프로토콜, 서브도메인, 도메인, 포트 중 하나만 달라도 CORS 에러가 난다.
        - `Access-Control-Allow-Origin: *`
            - 만약 주소를 일일이 지정하기 싫다면 *으로 모든 주소에 CORS 요청을 허용되지만 그만큼 보안이 취약해진다.
        - 유사한 헤더로 `Access-Control-Request-Method, Access-Control-Request-Headers, Access-Control-Allow-Methods, Access-Control-Allow-Headers` 등이 있다.

# 4. CORS

## 4.1 CORS 개요

CORS(Cross Origin Resource Sharing)는 Cross-Site HTTP Request를 가능하게 해주는 표준 규약이다.

### 4.1.1 배경

- 처음 전송되는 리소스의 도메인과 다른 도메인으로부터 리소스가 요청될 경우 해당 리소스는 cross-origin HTTP 요청에 의해 요청된다.
- 보안 상의 이유로, 브라우저들은 스크립트 내에서 초기화되는 cross-origin HTTP 요청을 제한한다.
- 하지만 지속적으로 웹 애플리케이션을 개선하고 쉽게 개발하기 위해서는 이러한 Request가 필요했다.
- 따라서 XMLHttpRequest가 cross-domain을 요청할 수 있도록 하는 방법으로 CORS가 탄생했다.

### 4.1.2 과정

- CORS 요청 시에는 미리 OPTIONS 주소로 서버가 CORS를 허용하는지 물어본다.
- 이때 Access-Control-Request-Method로 실제로 보내고자 하는 메서드를 알리고,
- Access-Control-Request-Headers로 실제로 보내고자 하는 헤더들을 알린다.
- Allow 항목들은 Request에 대응되는 것으로, 서버가 허용하는 메서드와 헤더를 응답하는데 사용된다.
- Request랑 Allow가 일치하면 CORS 요청이 이루어진다.

# 5. HTTP Method

HTTP 프로토콜을 이용해서 서버에 데이터(요청 정보)를 전달할 때 사용하는 방식

## 5.1 GET

### 5.1.1 개념

- 정보를 조회(SELECT)하기 위한 메서드이다. CRUD를 예로들 경우 R에 해당한다.

### 5.1.2 사용방법

- URL의 끝에 '?'가 붙고, 요청 정보가 'key=value' 형태의 쌍을 이루어 '? 뒤에 붙어 서버로 전송된다.
- 요청 정보가 여러 개일 경우 '&'로 구분한다.

### 5.1.3 특징

- URL에 요청 정보를 붙여서 전송한다.
- URL에 정보를 이어 붙이기 때문에 길이의 제한이 있다.
- 요청 정보를 사용자가 쉽게 눈으로 확인할 수 있다. 이로 인해 POST보다 보안에 취약하다.
- HTTP 패킷에 body는 비어 있는 상태로 전송한다.
- POST 방식 보다 빠르다. GET 방식은 캐싱을 사용할 수 있어, GET 요청과 그에 대한 응답이 브라우저에 의해 캐시된다.

## 5.2 POST

### 5.2.1 개념

- 서버의 값이나 상태를 바꾸기위한 용도의 메서드이다. CRUD 관점에서 C에 해당한다.
- 단 Update나 Delete에도 사용이 가능하다.

### 5.2.2 사용방법

- 요청 정보를 HTTP 패킨 안의 body에 숨겨서 서버로 전송한다.
- Request Header의 Content-Type에 해당 데이터 타입이 표현되며, 전송하고자 하는 데이터 타입을 적어주어야 한다.

### 5.2.3 특징

- Body 안에 숨겨서 요청 정보를 전송하기 때문에 대용량의 데이터를 전송하기에 적합하다
- 클라이언트 쪽에서 데이터를 인코딩하여 서버로 전송하고, 이를 받은 서버 쪽이 해당 데이터를 디코딩한다.
- GET 방식 보다 안전하다.

## 5.3 PUT

### 5.3.1 개념

- 새로운 리소스를 생성하거나, 대상 리소스를 나타내는 데이터를 수정하는데 사용한다.
- CRUP 관점에서 U에 해당한다.

### 5.3.2 특징

- 자원의 식별자를 미리 알고 있어야한다.
- 자원의 식별자가 존재하는 지는 영향이 없고, 자원이 존재하는 경우는 대체하고, 존재하지 않는 경우는 새로 생성한다.

### 5.3.3 POST와 차이

- POST는 Request 메세지에 포함된 엔티티를 이용해 새로운 자원을 생성한다.
- PUT은 Request 메세지에 함께 넘어온 식별자를 자원으로 만든다.
- POST는 동일한 요청이 2번 전달되면, 2개의 자원을 생성한다.
- PUT은 동일한 요청이 2번 전달되면, 첫 번째 요청에서 자원이 존재하지 않으면 새로 생성하고, 두 번째 요청에서는 자원이 이미 존재하기 때문에 대체한다.

## 5.4 DELETE

### 5.4.1 개념

- 지정한 리소스를 삭제한다.
- CRUD 관점에서 D에 해당한다.
- 어느 자원을 삭제할 지 URL에 드러난다.

## 5.5 PATCH

### 5.5.1 개념

- 리소스의 부분을 수정하는데 사용한다. 의미론적으로 U에 해당한다.

### 5.5.2 PUT과 차이

- PUT은 자원의 전체를 교체하지만, PATCH는 자원의 일부만을 교체하는 역할을 수행합니다.

