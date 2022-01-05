오늘은 CORS에 대한 개념과 Spring MVC와 Spring Security에서 CORS 설정하는 방법에 대해서 알아본다.

---

## CORS란 무엇인가?
CORS(Cross-Origin Resource Sharing)은 브라우저가 SOP(Same Origin Policy)를 우회하여 다른 출처(Origin)의 자원을 요청할 수 있도록하는 방법이다.

SOP는 Chorome, Edge와 같은 브라우저가 클라이언트의 교차 출처(Cross Origin) 요청을 막고, 동일 출처(Same Origin)에 대한 요청만 허가하는 정책을 말한다.

출처란 리소스의 위치를 의미하며 Scheme, Host, Port로 구성된다. 만약 Scheme, Host, Port 중 어느 하나라도 다르면 교차 출처로 인지하게된다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/147871342-0b7b78ae-5fb9-447e-8938-255758418d21.png width=500>
</p>

### CORS의 등장 배경

예전에는 웹 서비스의 크기가 크지 않아 다른 출처의 리소스를 사용할 일이 없었다. 하지만 웹 서비스가 커지면서 다른 출처의 서버가 호스팅하는 리소스를 사용할 일이 많아졌다. 하지만 브라우저는 CSRF, XSS 등 악의적인 사용자로 부터 공격을 방지하기 위해 SOP를 기본으로 했다.

이렇게 브라우저의 SOP를 우회해서 교차 출처의 자원을 요청하기위해 CORS가 나타난 것이다.

### Preflght Request

`Preflight`라는 단어에서 유추할 수 있듯이, 본격적인 클라이언트의 교차 출처 요청을 보내기전에 브라우저에서 서버가 CORS를 지원하는 지 확인하는 요청이다. 특정한 경우를 제외하고는 교차 출처 요청 전에는 항상 Preflight 요청이 선행한다.

Preflight는 클라이언트의 요청에 대한 헤더나 메서드같은 메타 정보를 서버에 보내고, 서버는 메타 정보를 가지고 브라우저의 요청을 허가할 지 결정한다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/147871586-38986dd4-8a70-48a1-8d6e-b1c62c87aaba.png width=500>
</p>

**Preflight 요청이 없다면?**

굳이 클라이언트의 요청을 보내기 전에 Preflight 요청을 보내는지에 대해 의문을 가질 수 있다. 이는 브라우저의 SOP가 나타나기 전에 존재했던 서버들을 지원하기 위함이다.

SOP가 나타나기 전에는 서버들이 별도의 CORS 설정을 하지 않았을 것이다. 따라서 Preflight 요청을 하지 않을 경우, 서버는 의심 없이 모든 클라이언트의 요청을 받아드릴 것이며, POST나 DELETE 같은 서버의 상태를 변경시키는 메서드로 인해 치명적인 문제에 빠질 수 있다.

따라서 Preflight 요청을 통해서 서버가 CORS를 지원하지 않는 경우 브라우저가 클라이언트의 요청이 전달되지 않도록 막는 것이다.

**서버는 Preflight 요청을 어떻게 구분할까?**

Preflight 요청의 가장 큰 특징은 다음 세 가지이다.

1. HTTP OPTIONS Method (RFC2616)
2. Origin Header 
3. Access-Control-Request-Method Header

OPTIONS 메소드나 Origin Header는 다른 요청에서도 사용할 수 있다. 하지만 `Access-Control-Request-Method`는 Preflight 요청에서만 추가되는 헤더로 서버는 이 세 가지를 종합하여 일반 요청과 Preflight 요청을 구분한다


<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/147874720-3671ff69-1f49-4bb8-8aa7-0c5ec7646ab1.png width=500>
</p>

### 교차 출처 요청에서 쿠키 사용하기

CORS는 기본적으로 Stateless하다. 따라서 사용자의 자격 정보 그러니까 쿠키와 같은 User Credentials를 기본적으로 사용하지 않는다.

CORS에서 쿠키를 사용하기 위해서는 `XMLHttpRequest`의 `withCredentials` 프로퍼티와 요청-응답의 `Access-Control-Allow-Credentials`를 사용한다.

예를들어 쿠키-세션으로 로그인 기능을 구현한 경우, 쿠키에 사용자 정보가 저장되기 때문에 인증 및 인가가 필요한 서비스에서는 반드시 사용자 요청 헤더에 쿠키가 포함되어야한다.

하지만 기본적으로 Preflight request에는 쿠키를 사용할 수 없고, 서버에서 쿠키를 허용하지 않는 경우에도 클라이언트의 요청에 쿠키를 포함할 수 없어 클라이언트와 서버 모두 쿠키를 사용할 수 있도록 설정해야한다.

**서버에서 설정**

서버에는 응답 헤더에 `Access-Control-Allow-Credentials`를 true로 설정하여 브라우저에게 쿠키를 허용하는 것을 알릴 수 있다.

클라이언트에서는 Preflight와 본 요청 헤더 모두 Access-Control-Allow-Credentials를 포함해야한다.

**클라이언트에서 설정**

클라이언트에서 쿠키를 사용하기 위해서는 `XMLHttpRequest` Object의 `withCredentials`를 true로 설정해야한다. 이는 요청에 쿠키가 포함되어있다는 의미이다.

따라서 서버와 클라이언트의 상호작용에 의해서 쿠키를 사용할 수 있는데 이를 표로 나타내면 아래와 같다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/147876817-c9979130-67e7-4d2e-a22c-3afe04862f90.png width=500>
</p>

## Spring에서 CORS 설정하기

### 초기 세팅

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/148204046-3bd3322e-9f67-4afb-b49f-12f02c83baf1.png width=500>
</p>

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/148204090-9818a82a-24ee-4781-98fe-bb2f36e1507f.png width=500>
</p>

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/148204466-adfa05fd-7353-404c-bd21-565a76f8a97e.png width=500>
</p>

초기 세팅은 위와 같고 요청 결과 서버에서 CORS 설정을 하지 않아 응답에 `Access-Control-Allow-Origin` 헤더가 없어 브라우저에서 응답을 클라이언트까지 보내지 않은 것을 알 수 있다.

### @CrossOrigin

스프링에서 제공하는 `@CrossOrigin` 어노테이션을 사용을 사용하면 간단하게 해결할 수 있다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/148209875-fe363a05-f389-4295-8fc0-a9ade5cf73f6.png width=500>
</p>

위 사진은 클래스 레벨에 `@CrossOrigin`을 달아줬기 때문에 전체 메소드에 적용될 것이다. 만약 메소드마다 다르게 설정하고 싶다면 메소드에 어노테이션을 달아주면된다.

### WebMvcConfigurer

글로벌하게 설정하기 위해서는 `WebMvcConfigurer`의 구현체를 만들어 `addCorsMappings`를 재정의 해주면된다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/148220538-01fbd852-40bc-4f8e-8309-65bc51ce18d1.png width=500>
</p>


### Spring Security에서 설정하기
만약 Spring Security 사용한다면 몇 가지 더 고려해야할 것이 있다.

첫 번째는 Csrf 필터를 비활성화 해야 한다. 디버깅을 해보면 알겠지만 Csrf 필터가 활성화 되어 있으면 Csrf 토큰이 없어 요청이 거부된다.

두 번쨰는 인가와 관련해서 Preflight 요청은 제외 해야 한다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/148226148-ed899b8f-a6ab-44ac-9e1a-4bd8c5406205.png width=500>
</p>


