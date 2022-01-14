## CORS?
---

### 등장 배경
예전에 웹 서비스를 개발할 때는 하나의 서버에서 브라우저의 모든 요청을 처리했다. 하지만 웹 서비스가 커지면서 브라우저가 다른 출처(혹은 출처)의 자원을 요청하게 되었다. 간단하게 생각하면 동일 출처에 요청한 것처럼 다른 출처의 자원을 요청하면 된다고 생각할 수 있지만 그렇지않다. 브라우저의 SOP(Same-Origin Policy)때문이다.

SOP는 브라우저가 다른 출처에 HTTP 요청을 할 수 없도록 제한한다. 서버간 통신에는 적용되지 않고 브라우저를 거칠 경우에만 적용되는 정책이다. 이는 악의적인 사용자가 소스 코드를 보고 CSRF(Cross-Site Request Forgery)나 XSS(Cross-Site Scripting) 같은 방법으로 정보를 탈취할 수 있기 때문이다.

SOP를 우회해서 다른 출처의 자원을 요청하는 방법이 CORS(Cross-Origin Resource Sharing)이다.

### 정의
CORS(Cross-Origin Resource Sharing)은 브라우저가 SOP를 우회하여 다른 출처(Origin)의 자원을 요청할 수 있도록하는 방법이다.

여기서 출처(Origin)은 scheme(protocol), host(domain), port로 구성되며, 동일 출처란 이 세가지 모두 동일한 것을 말한다.

브라우저와 서버는 HTTP 헤더를 사용하여 교차 출처 요청을 처리한다. 서버는 응답 헤더를 사용하여 API에 접근할 수 있는 클라이언트나 허용되는 메서드, 헤더, 쿠키를 표시할 수 있다.

아래 그림은 클라이언트-서버의 CORS 요청 처리 흐름이다.
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/147822387-04c4a4e7-ccb2-4edf-8685-a28263c70205.png>
</p>

1. Javascript client code를 통해 CORS 요청이 초기화된다
2. 서버에 요청을 보내기 전 헤더를 추가한다
3. 서버에서 요청이 허용되는지 여부를 나타내는 응답에 헤더를 포함한다
4. 요청이 허용되면 브라우저는 응답을 클라이언트 코드로 보낸다.


아래 그림은 CORS에 대한 요청과 응답 예제이다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/147823620-7aa55d28-611a-4a9c-ba1e-0acde20f03aa.png>
</p>

서버의 응답 헤더 중 CORS의 핵심은 `Access-Control-Allow-Origin:*`이다. 서버는 이 헤더를 교처 출처 요청이 허용됐다는 것을 알리기 위해 사용한다. Access-Control-Allow-Origin 헤더는 CORS 응답 헤더에 반드시 포함되어야 한다.

### 장점
- Wider audience
  
내가 Public API를 개발한다고 가정할 때, API를 사용하는 Python, Java 개발자는 Http Request 라이브러리를 사용해서 쉽게 자원을 요청할 수 있다.

하지만 Javascript 개발자는 SOP로 인해서 요청하기 쉽지 않지만, CORS를 통해 다른 출처의 자원을 요청할 수 있다.

- Servers stay in charge

서버로 하여금 Cross-origin 요청에 성공적으로 응답할 때 `Access-Control-Allow-Origin` 헤더를 반드시 포함하도록 하여 브라우저의 SOP가 주는 보안상의 이점을 위배하지 않는다.

- Flexibility

CORS는 서버에 cross-origin access 설정에 대해서 다음과 같은 유연성을 제공한다

    - 도메인
    - HTTP Method
    - HTTP headers
    - cookie data

- Easy for developers

서버에는 새로운 설정을 추가하도록하여, 클라이언트 개발자에게는 새로운 코드를 작성하지 않도록한다

- Reduced maintenance overhead

`<script>`태그 안에 cross-origin request 코드를 작성하는 JSONP와 같은 방법은 추가적인 코드를 요구하고, 표준도 아니기 때문에 개발자마다 다르게 구현할 수 있다.

반면에 CORS는 서버의 응답에 몇 가지 추가적인 헤더만 요구한다. 따라서 유지 보수에 따른 오버헤드가 적다

## Preflight request

브라우저가 서버에 요청이 가능한지 확인하는 것을 말한다. Preflight는 클라이언트의 요청에 대한 헤더나 메서드같은 메타 정보를 서버에 보내고, 서버는 메타 정보를 가지고 브라우저의 요청을 허가할 지 결정한다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/147871586-38986dd4-8a70-48a1-8d6e-b1c62c87aaba.png width=500>
</p>

**preflight request**는 브라우저의 SOP가 나타나기 전에 존재했던 서버들을 위해 존재한다. preflight의 결과 서버가 CORS를 지원하면 클라이언트의 요청에 정상적으로 응답할 수 있을 것이고, CORS를 지원하지 않으면 클라이언트의 요청을 브라우저에 의해서 서버에 전달되지 않을 것이다.

만약 preflight request가 존재하지 않을 경우, 서버는 의심없이 cross origin request에 응답할 것이고 이로 인해 POST, DELETE와 같은 서버의 상태를 변경시키는 요청에 의해서 치명적인 위험에 빠질 수 있을 것이다.

preflight request가 클라이언트의 요청와 구별되는 3가지 특징은 다음과 같다.

1. HTTP OPTIONS Method (RFC2616)
2. Origin Header 
3. Access-Control-Request-Method Header

HTTP OPTIONS Method는 서버-클라이언트의 통신과 관련된 정보를 조회할 때 사용되는 메서드이다.

OPTIONS 메서드와 Origin 헤더는 Preflight가 아니더라도 사용될 수 있는데, 이러한 요청과 구분하기 위해서 
`Access-Control-Request-Method`를 사용한다

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/147874720-3671ff69-1f49-4bb8-8aa7-0c5ec7646ab1.png width=500>
</p>

### Cookies
CORS는 기본적으로 Stateless하다. 따라서 사용자의 자격 정보 그러니까 쿠키와 같은 User Credentials를 기본적으로 사용하지 않는다.

CORS에서 쿠키를 사용하기 위해서는 `XMLHttpRequest`의 `withCredentials` 프로퍼티와 요청-응답의 `Access-Control-Allow-Credentials`를 사용한다.

예를들어 쿠키-세션으로 로그인 기능을 구현한 경우, 쿠키에 사용자 정보가 저장되기 때문에 인증 및 인가가 필요한 서비스에서는 반드시 사용자 요청 헤더에 쿠키가 포함되어야한다.

하지만 기본적으로 Preflight request에는 쿠키를 사용할 수 없고, 서버에서 쿠키를 허용하지 않는 경우에도 클라이언트의 요청에 쿠키를 포함할 수 없어 클라이언트와 서버 모두 쿠키를 사용할 수 있도록 설정해야한다.

**서버에서 설정**

서버에는 응답 헤더에 `Access-
Control-Allow-Credentials`를 true로 설정하여 브라우저에게 쿠키를 허용하는 것을 알릴 수 있다.

클라이언트에서는 Preflight와 본 요청 헤더 모두 Access-
Control-Allow-Credentials를 포함해야한다.

**클라이언트에서 설정**

클라이언트에서 쿠키를 사용하기 위해서는 `XMLHttpRequest` Object의 `withCredentials`를 true로 설정해야한다. 이는 요청에 쿠키가 포함되어있다는 의미이다.

따라서 서버와 클라이언트의 상호작용에 의해서 쿠키를 사용할 수 있는데 이를 표로 표현하면 아래와 같다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/147876817-c9979130-67e7-4d2e-a22c-3afe04862f90.png width=500>
</p>

JSONP와 같은 비표준 Cross-origin reqeust에서는 Preflight가 동작하지 않는다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/147876862-3836cec1-db6b-4fb1-94f2-9065c8e32d08.png width=500>
</p>

따라서 CSRF처럼 쿠키를 통한 악의적인 공격에 노출 될 수 있으므로 표준인 CORS를 따르는 것이 보안에 이점이 있다.


## 기타

### Origin

Origin은 Client-Resource의 위치를 말한다. CORS 요청일 경우 브라우저가 자동으로 헤더에 추가한다. Scheme, Host, Port 중 어느 하나라도 다르면 Cross-origin request이다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/147871342-0b7b78ae-5fb9-447e-8938-255758418d21.png width=500>
</p>

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/147871396-627cf706-8fc2-40f1-9048-ad46b77fc3ad.png width=500>
</p>


참고
---
- CORS in action
- https://developer.mozilla.org/ko/docs/Web/HTTP/CORS