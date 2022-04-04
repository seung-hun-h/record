# Apache HTTP Server, Apache Tomcat 연동
[연동 방법 참고](https://www.lesstif.com/system-admin/apache-httpd-tomcat-connector-mod_jk-reverse-proxy-mod_proxy-12943367.html)

## Tomcat Connector
Tomcat에서 제공하는 Apache와 연동하기 위한 모듈(mod_jk)이다. Apache JServ Protocol라는 특수한 프로토콜을 사용한다. HTTP에 비해 전송 속도가 빠르다는 장점이 있다. 

AJP를 지원하는 WAS로는 Apache Tomcat, Jetty, JBoss가 있다. ajp12, ajp13, ajp14 세 가지 버전 중 ajp13을 일반적으로 사용한다. 

## Apache Reverse Proxy
Apache를 Reverse Proxy로 사용하여 톰캣과 연동하고 mod_proxy를 모듈을 사용한다. mod_proxy는 톰캣과 연동할 떄 사용하는 가장 기본적인 모듈로 항상 로드 해주어야 한다. 그리고 프로토콜에 따라서 mod_proxy_http, mod_proxy_ajp 등이 있다.

### Forward Proxy
![image](https://user-images.githubusercontent.com/60502370/161453949-13b7e9c0-d006-4c37-a389-9849a72f7edf.png)

- 사용자가 naver.com에 요청했을 때, 포워드 프록시가 요청을 받아 naver.com에 접근해 결과를 사용자에게 전달
- Forward Proxy의 특징
  - Caching
    - 캐시 기능이 있어 자주 사용되는 컨텐츠라면 성능 향상이 월등함
  - Security
    - 서버측에 클라이언트 IP를 숨길 수 있다

### Reverse Proxy
![image](https://user-images.githubusercontent.com/60502370/161453963-68e39e78-e2fd-4f03-be51-10c2de8db9ac.png)

- 사용자가 naver.com에 요청할 경우 Reverse Proxy가 이 요청을 받아 내부 서버에 전달하고 결과를 받아 다시 사용자에게 전달
- Reverse Proxy의 특징
  - Load Balancing
  - Web acceleration
    - 유입, 유출되는 데이터를 압축하거나 캐싱할 수 있다
    - SSL 터널링을 제공할 수 있다
  - Sercurity & anonymity
    - 내부의 다양한 서버의 IP주소를 클라이언트로부터 숨길 수 있다


## mod_jk VS mod_proxy_ajp
Apache와 WAS를 연동할 때, WAS가 AJP를 제공한다면 HTTP 보다는 AJP를 사용하는 것이 데이터 전송 효율이나 속도 측면에서는 좋다고 한다.

일반적으로 두 가지 모듈 중 mod_jk를 사용하고, 현재는 두가지 방법의 성능차이도 미미하다고 한다. 따라서 기존에 mod_jk를 사용해서 연동했다면 굳이 mod_proxy_ajp로 전환할 필요는 없을 것 같다.

mod_proxy_ajp가 모듈을 별도로 설치할 필요도 없고 http등 다른 프로토콜로의 전환도 빠르게할 수 있기 때문에, 만약 프로젝트를 처음 시작하는 경우에는 굳이 mod_jk를 사용하기 보다는 mod_proxy_ajp를 사용하는 것이 좋을 것 같다.

다만 서비스가 커지게되면 mod_proxy_ajp에서 성능 유지를 위한 설정이 조금 복잡해질 수 있다고 하기 때문에 그떄가서 mod_jk로 전환하면 될 것이다.

## AJP를 사용한 Tocat - Apache 연동 이슈
### secretRequired 이슈 
- https://nirsa.tistory.com/131

- 최종 설정
```xml
<Connector protocol="AJP/1.3" address="0.0.0.0" secretRequired="false" port="8009" redirectPort="8443" />
```

### 참고
- https://nowonbun.tistory.com/679
- https://cwiki.apache.org/confluence/display/TOMCAT/Connectors
