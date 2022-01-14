# Servlet
- 클라이언트의 요청을 처리하고, 그 결과를 반환하는 Servlet 클래스의 구현 규칙을 지킨 자바 웹 프로그래밍 기술
- 클라이언트의 HTTP 요청에 대해 특정 기능을 수행한다.
- HTML 문서를 생성하거나 JSON과 같은 데이터를 생성하여 응답한다.

일반적인 웹 서버는 정적인 리소스만 제공한다. 따라서 클라이언트가 동적인 리소스에 대한 요청을 했을 때 WAS에 요청을 위임한다. 이때 동적인 리소스를 제공해주기 위해 도와주는 컴포넌트가 서블릿이다.

## 동작 방식
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/132290700-712b8142-e767-4a37-9308-377edc8fd3af.png>
</p>

1. 클라이언트가 서버에 Http Request를 전달한다
2. 요청을 전달 받은 컨테이너는 HttpServletRequest, HttpServletResponse 객체를 생성한다
3. URL에 매핑된 서블릿을 찾는다
4. 해당 서블릿에서 service 메소드를 호출한다
5. 요청을 처리한 서블릿은 HttpServletResponse객체에 응답을 전달한다
6. 컨테이너는 HTTP response를 전달하고 HttpServletResponse, HttpServletRequest 객체는 소멸한다

## 서블릿은 자바로 구현된 CGI(Common Gateway Interface)
- 웹 서버에서는 동적인 요청을 처리하지 못하기 때문에 동적인 처리는 컨테이너에게 위임한다.
- 웹 서버와 컨테이너의 종류는 매우 다양하기 때문에 일관된 데이터 교환 방식이 필요하다
- 웹 서버 - 컨테이너간 데이터 교환 규칙이 CGI이다.

CGI는 특별한 라이브러리나 도구를 의미하는 것은 아니고, 별도로 제작된 웹 서버와 프로그램간의 교환 방식이다. CGI는 어떤 프로그래밍 언어로도 구현이 가능하며, 별도로 만들어 놓은 프로그램에 HTML의 Get or Post 방법으로 클라이언트의 데이터를 환경변수로 전달하고 프로그램의 표준 출력 결과를 클라이언트에게 전송하는 것이다. 즉, 자바 어플리케이션 코딩을 하듯 웹 브라우저용 출력화면을 만드는 방법이다.

## HTTP 프로토콜을 이용한 서버와 클라이언트의 통신과정
클라이언트는 정보를 얻기 위해 서버로 HTTP 요청을 전송하고, 서버는 이를 해석하여 정적 자원에 대한 요청일 경우 자원을 반환해주고, 그렇지 않은 경우 CGI 프로그램을 실행시켜 해당 결과를 리턴해준다. 이때 서버는 CGI 프로그램에게 요청을 전달해주고, 결과를 전달받기 위한 파이프라인을 연결한다. 그래서 CGI 프로그램은 입력에 대한 서비스를 수행하고, 결과를 클라이언트에게 전달하기 위해 결과 페이지에 해당하는 MIME 타입의 컨텐츠데이터를 웹 서버와 연결된 파이프라인에 출력하여 서버에 전달한다. 서버는 파이프라인을 통해 CGI 프로그램에서 출력한 결과 페이지의 데이터를 읽어, HTTP 응답헤더를 생성하여 데이터를 함께 반환 해준다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/132292685-1f1a393d-5fff-42ab-bd88-49b26316dc4a.png>
</p>

# Servlet Container
- 서블릿을 관리해주는 컨테이너
- 서블릿의 생명주기를 관리하고 요청에 따른 스레드를 생성해준다
- 클라이언트의 Request를 받아주고 Reponse를 보낼수 있게 서버와 소켓을 만들어서 통신을 해준다.

## 통신 지원
서블릿과 웹 서버가 통신할 수 있는 손쉬운 방법을 제공한다. 소켓을 만들고 특정 포트를 리스닝하고, 연결 요청이 오면 스트림을 생성해서 요청을 받는데, 이러한 과정을 컨테이너가 대신해준다. 서블릿 컨테이너는 이런 통신 과정을 API로 제공하고 있어 손쉽게 사용이 가능하다

## 생명 주기 관리
서블릿 컨테이너가 가동되는 순간 서블릿 클래스를 로딩해서 인스턴스화하고, 초기화 메서드를 호출하고, 요청이 들어오면 적절한 서블릿 메소드를 찾아서 호출한다.

## 멀티 스레딩 관리
서블릿 컨테이너는 해당 서블릿의 요청이 들어오면 스레드를 생성해서 작업을 수행한다.

## 선언적 보안관리
서블릿 컨테이너는 보안 관련된 기능을 제공한다. 따라서 서블릿 내부에 보안과 관련된 메소드를 구현하지 않아도 된다.

## 서블릿의 생명주기
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/132293858-2e7ff2ea-7104-49a9-bd7f-af6daaf9598e.png>
</p>

1. 클라이언트의 요청이 들어오면 컨테이너는 해당 서블릿에 메모리에 있는지 확인하고, 없을 경우 init() 메소드를 호출하여 적재한다. 실행 중에 서블릿이 변경될 경우, 기존 서블릿을 파괴하고 init()을 통해 새로운 내용을 다시 메모리에 적재한다
2. init() 호출 후 클라이언트의 요청에 따라서 service() 메소드를 통해 doGet(), doPost()로 분기된다.
3. 컨테이너가 서블릿에 종료 요청을 하면 destroy() 메소드가 호출되는데 마찬가지로 한 번만 실행되며, 종료시에 처리해야할 작업들은 destroy() 메소드를 오버라이딩하여 구현한다.

# Servlet Context
- 하나의 서블릿이 서블릿 컨테이너와 통신하기 위해서 사용되는 메서드를 가지고 있는 인터페이스이다. 
- servlet context를 통해 서블릿은 웹 애플리케이션의 raw input stream이나 application scope의 객체, logging과 관련된 정보에 접근할 수 있다.

ServletContext는 ServletConfig 인터페이스를 통해 얻을 수 있다. 자바의 서블릿은 HttpServlet을 상속받고 있는데 HttpServlet는 Servlet과 ServletConfig를 구현하고 있어, getServletContext()를 통해 ServletContext를 얻을 수 있다.

# Servlet Listner
웹 어플리케이션에서 발생하는 주요 이벤트를 감지하고 각 이벤트에 대한 특별한 작업이 필요한 경우에 사용할 수 있다.
- 서블릿 컨텍스트 수준의 이벤트
  - 컨텍스트 라이프 사이클 이벤트
  - 컨텍스트 애트리뷰트 변경 이벤트
- 세션 수준의 이벤트
  - 세션 라이프 사이클 이벤트
  - 세션 애트리뷰트 변경 이벤트

## ContextLoaderListner
- ServletListner의 구현체
- WebApplicationContext를 서블릿 컨텍스트 라이프 사이클에 따라 등록하고 소멸시켜준다
- 서블릿에서 IoC 컨테이너를 ServletContext를 통해 꺼내 사용할 수 있다.

# Servlet Filter
- Request 객체를 서블릿으로 보내고, 서블릿이 작성한 응답을 클라이언트로 보내기 전에 특별한 처리가 필요한 경우에 사용할 수 있다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/132297833-2351cb06-7e62-44c9-85c3-5e00c035dd80.png>
</p>

- https://mangkyu.tistory.com/14?category=761303
- https://jordy-torvalds.tistory.com/14
- https://jusungpark.tistory.com/15
- https://velog.io/@seanlion/cgi
- https://www.informit.com/articles/article.aspx?p=170963&seqNum=6
- https://www.geeksforgeeks.org/difference-between-servletconfig-and-servletcontext-in-java-servlet/