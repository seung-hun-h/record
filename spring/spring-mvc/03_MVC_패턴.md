# MVC 패턴
## 서블릿과 JSP

```java
@WebServlet(name = "memberFormServlet", urlPatterns="/servlet/members/new-form")
public class MemberFormServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) {
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");
        PrintWriter w = response.getWriter();

        w.write("<!DOCTYPE html>\n" +
                "<html>\n" +
                "<head>\n" +
                "    <meta charset=\"UTF-8\">\n" +
                "    <title>Title</title>\n" +
                "</head>\n" +
                "<body>\n" +
                "<form action=\"/servlet/members/save\" method=\"post\">\n" +
                "    username: <input type=\"text\" name=\"username\" />\n" +
                "    age:      <input type=\"text\" name=\"age\" />\n" +
                " <button type=\"submit\">전송</button>\n" + "</form>\n" +
                "</body>\n" +
                "</html>\n");
    }
}
```

- 서블릿을 사용하면 자바 코드로 동적인 HTML을 만들 수 있다
- 하지만 서블릿을 사용하여 String으로 HTML을 만드는 것은 매우 비효율적이다
- 자바 코드로 HTML을 만들어 내는 것 보다, HTML 코드에 자바 코드를 작성하는 것이 더 편리할 것이다
- 이러한 이유로 템플릿 엔진이 나타났다
  - JSP, Thymeleaf, Freemarker, Velocity 등이 존재한다

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>Title</title>
  </head>
<body>
<form action="/jsp/members/save.jsp" method="post"> username: <input type="text" name="username" /> age: <input type="text" name="age" /> <button type="submit">전송</button>
</form>
  </body>
  </html>
```

- JSP는 HTML 형식으로 작성되어 있고, 자바 코드도 작성이 가능하다
- JSP를 사용하면 동적인 부분에 자바 코드를 사용하여 편리하게 HTML을 작성할 수 있다
- 하지만 JSP에는 너무 많은 책임이 부여되어 있다
  - HTML을 보여주는 뷰의 영역과
  - 데이터 조회 등 다양한 자바 코드가 노출되어 있다
- 비즈니스 로직은 서블릿에 작성하고, JSP는 목적에 맞게 화면에 그리는 일에 집중하도록 하는 것이 좋다

## MVC 패턴 - 개요
- 너무 많은 역할
  - 하나의 서블릿이나 JSP가 비즈니스 로직과 뷰 렌더링까지 모두 처리함현 너무 많은 역할을 하게되고 유지보수가 어려워진다
- 변경의 라이프사이클
  - UI를 수정하는 일과 비즈니스 로직을 수정하는 일은 다른 시점에 일어날 가능성이 높다
  - 두 가지 작업의 라이프사이클이 다르다
- 기능 특화
  - JSP와 같은 템플릿 엔진은 화면을 렌더링하는데 최적화되어 있기 때문에 이 부분의 업무만 담당하는 것이 좋다
- Model, View, Controller
  - Controller: HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다. 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다
    - 컨트롤러에 비즈니스 로직을 둘 수 있지만, 이렇게되면 컨트롤러가 너무 많은 역할을 담당한다
    - 일반적으로 비즈니스 로직은 서비스라는 계층을 별도로 만들어서 처리한다
    - 컨트롤러는 비즈니스 로직에 있는 서비스를 호출하는 역할을 담당한다
  - Model: 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 뷰는 비즈니스 로직이나 데이터 접근으 몰라도 되고, 화면을 렌더링하는 일에 집중할 수 있다
  - View: 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다
  
  ### MVC 패턴1

  ![Screen Shot 2022-07-20 at 9 41 34 AM](https://user-images.githubusercontent.com/60502370/179871460-8850a870-63fd-4d26-954e-c590561f0eb6.png)
  
  ### MVC 패턴2

  ![Screen Shot 2022-07-20 at 9 42 38 AM](https://user-images.githubusercontent.com/60502370/179871538-f48cf9bd-ff69-4951-8957-16b2ce3e20ed.png)

```java
@WebServlet(name = "mvcMemberListServlet", urlPatterns = "/servlet-mvc/
  members")
public class MvcMemberListServlet extends HttpServlet {
    private MemberRepository memberRepository = MemberRepository.getInstance();
        @Override
        protected void service(HttpServletRequest request, HttpServletResponse response)throws ServletException, IOException {
            System.out.println("MvcMemberListServlet.service");
         
            List<Member> members = memberRepository.findAll();
            request.setAttribute("members", members);
         
            String viewPath = "/WEB-INF/views/members.jsp";
         
            RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
            dispatcher.forward(request, response);
        }
}
```

> redirect vs forward
> redircet는 실제 클라이언트에 응답이 나갔다가, 클라이언트가 해당 경로로 다시 요청한다. 따라서 클라이언트가 인지할 수 있고 URL 경로도 실제로 변경된다.<br>
> forward는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 인지하지 못한다

## MVC 패턴 - 한계
- MVC 패턴을 적용하면 컨트롤러와 뷰의 역할을 명확하게 구분할 수 있다
- 하지만 컨트롤러에 코드의 중복이 많고, 사용하지 않는 코드도 생긴다

### 포워드 중복
- viewPath로 이동하는 코드가 항상 중복되어 호출한다

```java
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.foward(request, response);
```

### ViewPath 중복
```java
String viewPath = "/WEB-INF/views/new-form.jsp";
```

- prefix: `/WEB-INF/views`
- suffix: `.jsp`
- jsp가 아닌 다른 템플릿 엔진을 사용하면 코드를 전부 다 바꿔야 한다

### 사용하지 않는 코드
```java
HttpServletRequest request, HttpServletResponse response
```

- request, response를 둘 다 사용할 때도 있고, 사용하지 않을 떄도 있다
- HttpServletRequest, HttpServletResponse를 사용하는 코드는 테스트하기도 어렵다

### 공통처리가 어려움
- 기능이 복잡해질 수록 컨트롤러에서 공통적으로 처리해야 할 부분이 많아진다
- 공통 메서드를 만든다고 해도, 해당 메서드를 호출하는 것 자체가 중복이다