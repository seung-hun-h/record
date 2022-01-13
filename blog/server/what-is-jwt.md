오늘은 토큰 기반 인증 시스템에서 가장 대중적으로 사용하는 JSON Web Token(JWT)에 대해서 알아본다

## JWT란?
---
**JSON Web Token(JWT)**는 JSON Obejct 형태로 클라이언트-서버간 정보를 안전하게 전송하기 위한 표준이다. 간단하고 독립적인 방법을 제공하며 HMAC 같은 암호화 알고리즘이나 private/public key를 통해 서명(sign)된다. 주로 로그인 기능에서 사용된다.

사용자가 로그인에 성공하게 되면 서버에서는 JWT를 발급한다. 클라이언트에서는 발급 받은 토큰을 쿠키, 로컬 스토리지 같은 저장소에 보관한다. 그리고 서버에 요청할 때마다 토큰을 함께 전송하는데 쿠키나 HTTP URI 혹은 헤더를 사용하는데 일반적으로는 아래처럼 `Authorization` 헤더에 토큰을 담아 요청과 함께 전송한다.

`Authorization: Bearer <token>`

JWT는 Stateless한 인증 매커니즘이므로 서버에서는 클라이언트의 상태를 저장하지 않아도된다. 그리고 헤더를 사용할 경우 쿠키를 사용하지 않기 때문에 CORS에 대한 이슈도 해결할 수 있다. JWT를 사용한 인증 흐름은 아래 그림과 같다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/149324262-778f08ec-49e2-4f35-9a11-83f7fc327973.png width=400>
</p>   

## JWT의 구조
---
JWT는 `점(.)`으로 구분되어 Header, Payload, Signature 세 가지 부분으로 나뉜다. JWT는 일반적으로 `xxxx.yyyy.zzzz`와 같은 형식을 따른다.

### Header
헤더는 `alg`, `typ`로 나뉜다. alg는 서명(Sign) 알고리즘이고, typ는 토큰의 유형을 의미한다.

```json
{
    "alg": "HS256",
    "typ": "JWT"
}
```

### Payload
Payload에는 클레임(Cliam)이 담겨있다. 클레임이란 사용자나 추가적인 데이터들에 대한 조각들이다. 클레임은 registered, public, private 세 가지 유형으로 나뉜다.

- Registered claims: 미리 정의된 클레임들로 필수는 아니지만 사용하는 것이 권장된다. JWT의 핵심 목표 중 하나인 간명한 표현을 위해 세자로 줄임말이 사용된다
  - iss: 토큰 발급자(issuer)
  - sub: 토큰 제목(subject), 유일해야한다.
  - aud: 토큰 대상자(audience)
  - exp: 토큰 만료시간(expiration)
  - nbf: 토큰 활성 날짜(not before)
  - iat: 토큰 발급 시간(issued at)
  - jti: 토큰 식별자 (JWT ID)

- Public claims: 사용자 정의 클레임으로 공개용 정보 전달을 위해 사용된다. 충돌 방지를 위하여 URI를 사용한다
- Private claims: 사용자 정의 클레임으로 클라이언트-서버간 정보 공유를 위해 사용한다.

```json
{
    "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

### Signature
서명(Signature)를 생성하기 위해서는 Base64URL로 인코딩된 헤더와 페이로드를 비밀 키와 함께 헤더에 정의된 암호화 알고리즘을 사용해서 해싱한다.

HMAC SHA256 알고리즘을 사용한 예는 아래와 같다.
```java
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

### 예시
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/149329119-afc53d01-0aa0-47b1-85b8-b95e969a3af9.png width=500>
</p>   
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/149329192-79f871e3-9605-45d8-afdb-c8b2438a8721.png width=500>
</p>   

## JWT의 장단점

### 확장성
JWT가 Stateless 하다는 것은 서버에서 클라이언트의 상태를 저장하지 않는 것이다. 서버에서는 단순히 토큰의 유효성만 검사할 뿐이다. 따라서 웹 애플리케이션의 수평적 확장에 용이하다.

### 보안
클라이언트에서 보통 JWT를 로컬 스토리지나 쿠키에 저장한다. 자바스크립트는 같은 도메인에서 웹 스토리지에 접근이 가능하다. 그러므로 JWT는 XSS 같은 자바스크립트 코드를 주입하는 공격에 취약할 수 있기 때문에 사용자의 민감 정보를 저장해서는 안된다.

### RESTful API
현대 웹 애플리케이션은 대부분 RESTful API를 사용한다. RESTful API의 가장 큰 특징은 서버가 클라이언트의 상태를 저장하지 않는다는 것이다. 세션 기반의 인증은 사용자의 인증 상태를 저장하지만 JWT는 그렇지 않다. 따라서 RESTful API를 사용한다면 JWT가 좋은 선택이 될 수 있다.

만약 브라우저를 통한 Cross-orgin request가 이루어진다면 CORS를 사용해야한다. 하지만 쿠키는 동일한 도메인에서만 사용이 가능하다(별도의 설정을 하면 사용 가능 하다). 하지만 JWT는 API를  Stateless하게 유지하기 때문에 CORS에 대해 고려해야할 사항이 줄어든다.

### 성능
세션 기반의 인증에서는 쿠키에 세션 아이디만 담고 있으면된다. 하지만 JWT에서는 다양한 정보를 Base64URL로 인코딩한다. JWT의 크기가 커지면 HTTP 요청에 대한 오버헤드가 커질 수 있다. 따러서 JWT에는 최소한의 정보만 보관해야한다.

### 로그아웃
세션에는 클라이언트의 상태를 저장하기 때문에 사용자의 로그아웃 처리가 용이하다. 하지만 JWT는 Stateless 하기 때문에 사용자의 로그아웃 처리가 어렵다.