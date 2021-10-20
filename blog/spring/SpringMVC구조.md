오늘은 Spring MVC의 구조에 대해 알아본다.

# MVC 패턴
---
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/133188404-8eb2498d-4906-48df-baa7-05c0cf173d17.png width=600>
</p>
MVC 패턴은 웹 애플리케이션의 구조를 Model-View-Controller로 역할을 나눈 디자인 패턴을 의미한다. 먼저 MVC 패턴의 등장 배경에 대해서 알아본다.

## MVC 등장 배경
---

상품 관리 서비스를 개발한다고 가정하자. 상품 관리 서비스에는 상품 등록, 수정, 삭제가 있다.

### MVC 패턴을 적용하지 않았을 때
MVC 패턴을 적용하지 않고 서블릿만을 사용하여 상품 등록 뷰를 클라이언트에게 전달하기 위해서는 Response Body에 직접 HTML를 작성하여 응답해야할 것이다. 그리고 아래와 같이 등록할 상품 정보를 전달받아 DB에 저장하는 역할의 서블릿도 저장 결과를 클라이언트에게 전달하기 위해 서블릿에서 직접 HTML을 작성해야한다.

```java
@WebServlet(name = "itemSaveServlet", urlPatterns = "/items/save")
public class itemSaveServlet extends HttpServlet {

    private final ItemRepository itemRepository = ItemRepository.getInstance();


    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String itemName = request.getParameter("itemName");
        int price = Integer.parseInt(request.getParameter("price"));
        int stock = Integer.parseInt(request.getParameter("stock"));

        
        Item item = new item(itemName, price, stock);
        itemRepository.save(item);

        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter w = response.getWriter();
        w.write("<html>\n" +
                "<head>\n" +
                " <meta charset=\"UTF-8\">\n" +
                "</head>\n" +
                "<body>\n" +
                "<ul>\n" +
                " <li>id="+item.getId()+"</li>\n" +
                " <li>Item Name="+item.getItemName()+"</li>\n" +
                " <li>Price="+item.getPrice()+"</li>\n" +
                " <li>Stock="+item.getStock()+"</li>\n" +
                "</ul>\n" +
                "<a href=\"/index.html\">메인</a>\n" +
                "</body>\n" +
                "</html>");
    }
}
```

HTML을 직접 개발자가 작성하면 생산성이 매우 떨어진다. 이로인해 HTML과 유사한 JSP가 나타난다.

### JSP의 등장
정적인 HTML은 동적인 회원의 저장, 삭제, 수정 결과를 나타낼 수 없다. 동적인 결과를 나타내기위해 HTML의 필요한 곳에 자바 코드를 작성하려는 시도가 있었고 그 결과 JSP가 탄생하였다. JSP를 사용하면 손쉽게 HTML을 작성할 수 있고 필요에 따라 자바 코드를 작성하여 클라이언트의 동적인 요청을 처리할 수 있다.
```jsp
<%@ page import="hun..domain.item.ItemRepository" %>
<%@ page import="hun.domain.item.Item" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
ItemRepository itemRepository = ItemRepository.getInstance();

String itemName = request.getParameter("itemName");
int price = Integer.parseInt(request.getParameter("price"));
int stock = Integer.parseInt(request.getParameter("stock"));
Item item = new item(itemName, price, stock);
itemRepository.save(item);
%>
<html>
    <head>
        <meta charset="UTF-8">
    </head>
    <body>
        <ul>
            <li>id=<%=item.getId()%></li>
            <li>itemName=<%=item.getItemName()%></li>
            <li>price=<%=item.getPrice()%></li>
            <li>stock=<%=item.getStock()%></li>
        </ul>
        <a href="/index.html">메인</a>
    </body>
</html>
```

JSP에 자바 코드를 작성하면 HTML 형식의 데이터를 손쉽게 작성할 수 있고, 서블릿은 단순히 요청이 들어오면 해당 서블릿에 렌더링만 해주면된다. 서블릿만 사용했을떄 보다는 많이 발전했지만 문제는 여전히 존재한다.<br/>

JSP에 너무 많은 책임이 부여된다. JSP에는 비즈니스 로직과 뷰에 대한 책임이 혼재한다. 비즈니스 로직을 수정 했을 때, 뷰에 대한 수정이 필요할 때 모두 JSP 코드를 수정해야한다. 그리고 결정적으로 **비즈니스 로직과 뷰는 변경의 라이프 사이클이 다르다**. 즉 비즈니스 로직의 변경 시기와 뷰의 변경 시기가 서로 다르다는 것이다. 보통 변경의 라이프 사이클이 다를 경우 역할을 분리해주는 것이 좋다.<br/>

### MVC 패턴의 등장
MVC 패턴은 이전에 JSP나 서블릿으로만 처리했던 과정을 모델-뷰-컨트롤러로 역할을 나누는 것을 의미한다.<br/>
- 모델: 애플리케이션의 데이터를 나타낸다. 그리고 이러한 데이터를 가공하는 책임을 가진다
- 뷰: 사용자 인터페이스 요소를 나타낸다. 즉 데이터 및 객체의 입력 그리고 출력을 담당한다. 데이터를 기반으로 사용자들이 볼 수 있는 화면을 의미한다
- 컨트롤러: 데이터와 뷰의 요소들을 잇는 다리역할을 한다. 사용자가 뷰를 통해 데이터를 추가 및 변경하면 이에 대한 이벤트를 처리하는 컴포넌트이다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/133192499-b7e4a394-cd45-42e0-9ef2-680a849812ff.png width=600>
</p>


```java
@WebServlet(name = "itemSaveServlet", urlPatterns = "/items/save")
public class itemSaveServlet extends HttpServlet {

    private final ItemRepository itemRepository = ItemRepository.getInstance();


    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String itemName = request.getParameter("itemName");
        int price = Integer.parseInt(request.getParameter("price"));
        int stock = Integer.parseInt(request.getParameter("stock"));

        
        Item item = new item(itemName, price, stock);
        itemRepository.save(item);

        request.setAttribute("item", item);

        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher requestDispatcher = request.getRequestDispatcher(viewPath);
        requestDispatcher.forward(request,response);
    }
}
```
처음 봤었던 상품 등록 서블릿에 MVC 패턴을 적용한 결과이다. 이전과 달라진 점은 HttpServletRequest에 데이터를 저장하고 지정된 뷰 경로로 포워딩한다는 것이다. HttpServletRequest의 `setAttribute()`를 사용하면 현재 요청 스코프에 대해서 데이터를 저장할 수 있다.

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <meta charset="UTF-8">
    </head>
    <body>
        <ul>
            <li>id=<%=item.getId()%></li>
            <li>itemName=<%=item.getItemName()%></li>
            <li>price=<%=item.getPrice()%></li>
            <li>stock=<%=item.getStock()%></li>
        </ul>
        <a href="/index.html">메인</a>
    </body>
</html>
```

그리고 JSP에는 비즈니스 로직이 사라지고 뷰에 대한 책임만 수행하는 것을 확인할 수 있다.

## MVC 패턴의 한계
---

지금은 아주 단순한 서비스이지만 만약 서비스가 커지게 되면 문제점을 여럿 확인할 수 있다.

```java
@WebServlet(name = "itemSaveServlet", urlPatterns = "/items/save")
public class itemSaveServlet extends HttpServlet {

    private final ItemRepository itemRepository = ItemRepository.getInstance();


    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException { // 3
        ...
        
        String viewPath = "/WEB-INF/views/save-result.jsp"; // 1
        RequestDispatcher requestDispatcher = request.getRequestDispatcher(viewPath); // 2
        requestDispatcher.forward(request,response);
    }
}
```

1. 뷰에 대한 경로가 중복된다
2. 포워딩에 대한 코드가 중복된다
3. 사용하지 않는 객체를 인자로 전달 받는다

서블릿이 수백개가 생기면 뷰에 대한 경로와 포워딩 코드를 모두 중복해서 작성해야한다. 그리고 만약 뷰 경로가 변경되면 모든 서블릿에 작성된 뷰 경로를 수정해야할 것이다. 또한 현재는 Request, Response 객체를 모두 사용하고 있지만 만약 단순히 뷰를 렌더링하는 서블릿이라면 Response 객체는 전혀 사용하지 않을 것이다.<br/>

이러한 MVC 패턴의 한계를 극복하기 위하여 프론트 컨트롤러 패턴이 등장하였다.

# FontController 패턴
---
MVC 패턴을 사용하면 기능이 복잡해질 수록 컨트롤러에서 공통으로 처리하는 부분이 많아진다. 만약 공통 처리 부분이 지나치게 많아지면 컨트롤러의 핵심적인 책임을 수행할 수 없게 될 수도 있다. 이러한 문제를 해결하기 위해서는 공통 처리를 처리할 클래스가 필요할 것이다.<br/>

이러한 필요에 의해 컨트롤러의 앞 단에 현관문 역할을 하는 기능을 추가한 프론트 컨트롤러 패턴이 등장하였다.

**변경 전**
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/133195139-9cd138a3-cfe6-4e5b-a679-ef1bc644bc17.png width=600>
</p>


**변경 후**
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/133195266-92d48952-6e5b-45ba-b85f-6555d70b166f.png width=600>
</p>

## FontController 패턴 적용

모든 코드를 작성할 수 없어 코드의 일부분만 발췌하여 작성한다.

```java
public interface Controller {

    ModelAndView process(Map<String, String> paramMap);
}
```
먼저 컨트롤러 인터페이스를 선언한다. 각 컨트롤러의 HttpServletRequest, HttpServletResponse 에 대한 의존을 없애기 위해 데이터를 담고있는 Map 객체를 인자로 받는다.

```java
public class ItemSaveController implements Controller {

    private ItemRepository itemRepository = ItemRepository.getInstance();

    @Override
    public ModelAndView process(Map<String, String> paramMap) {

        String itemName = paramMap.get("itemName");
        int price = Integer.parseInt(paramMap.get("price"));
        int stock = Integer.parseInt(paramMap.get("stock"));

        Item item = new item(itemName, price, stock);
        itemRepository.save(item);

        ModelAndView modelAndView = new ModelAndView("save-result"); // 1 
        modelAndView.setAttribute("member", member); // 2
        return modelAndView;
    }
}
```
그리고 상품 등록 요청을 처리할 컨트롤러를 구현한다. 컨트롤러는 요청 처리 결과로 데이터와 렌더링할 뷰의 논리 경로를 담고있는 ModelAndView 객체를 반환한다.<br/>

프론트 컨트롤러 패턴을 사용하여 MVC 패턴의 한계를 해결하였다. 컨트롤러는 더이상 HttpServlet을 상속하지 않고 요청을 처리하기 위해 데이터를 저장하고 사용할 수 있도록 Map 객체를 인자로 받는다.<br/>

HttpServlet을 상속하지 않기 때문에 더이상 HttpServletRequest, HttpServletResponse 객체를 사용하지 않는다. 그리고 뷰의 상세한 경로가 아닌 논리 경로만 ModelAndView 객체로 전달하기 때문에 뷰의 경로가 변경될 경우 변경 지점이 한 곳으로 제한될 수 있다.

```java
@WebServlet(name = "frontController", urlPatterns = "/items/*")
public class FrontController extends HttpServlet {

    private Map<String, Controller> controllerMap = new HashMap<>(); // 1

    public FrontControllerServlet() {
        controllerMap.put("/items/new-form", new ItemFormController());
        controllerMap.put("/items/save", new ItemSaveController());
        controllerMap.put("/items", new ItemListController());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String requestURI = request.getRequestURI();

        Controller controller = controllerMap.get(requestURI); // 2
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        Map<String, String> paramMap = createParamMap(request);
        ModelAndView modeAndView = controller.process(paramMap); // 3
        String viewName = modelAndView.getViewName();
        View view = new View("/WEB-INF/views/" + viewName + ".jsp")// 4
        view.render(mv.getModel(), request, response); // 5
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```
1. 경로에 따른 컨트롤러를 담고있다
2. 클라이언트의 요청 URI를 통해 처리할 컨트롤러를 가져온다
3. URI의 파라미터 데이터를 Map 객체에 담고 컨트롤러에 전달한다
4. 컨트롤러가 반환한 ModelAndView 객체에 담겨있는 뷰의 논리적 경로를 가져와 렌더링할 뷰의 경로를 완성시킨다
5. 해당 뷰로 렌더링한다.

프론트 컨트롤러에는 이전에 모든 컨트롤러가 수행했던 공통 기능을 담당하고있다. 이를 통해 각 컨트롤러는 자신의 핵심적인 기능에 집중할 수 있게 되었고, 공통 기능에 변경이 발생하면 프론트 컨트롤러만 변경하면된다.<br/>

Spring MVC의 구조는 더욱 복잡하지만 기본적으로 이러한 구조를 따르고 있다.

# Spring MVC
---
Spring MVC는 디자인 패턴을 적용하여 잘 설계되어있다. 대표적으로 위에서 본 것과 같이 프론트 컨트롤러 패턴이 적용되어있고, 다양한 형태의 컨트롤러를 제공하기 위해서 어답터 패턴도 적용되어있다.
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/133201605-87be08f9-3cfc-414a-aa6e-ed36db93d71a.png width=600>
</p>

먼저 각 컴포넌트들을 알아보고 전체적인 흐름을 정리한다.

## Spring MVC Component

### DispatcherServlet
가장 앞단에서 HTTP 프로토콜러 들어오는 모든 요청을 가장 먼저 받아 적합한 컨트롤러에 요청을 위임해주는 프론트 컨트롤러이다.<br/>

DispatcherServlet이 등장함에 따라 web.xml의 역할이 상당히 축소되었다. 기존에는 서블릿에 대해 매핑될 URL을 모두 web.xml에 등록 해야 했다. 하지만 dispatcher-servlet을 web.xml에 등록하여 애플리케이션으로 들어오는 모든 요청을 핸들링하고 공통 작업을 처리할 수 있게 되었다.<br/>

하지만 DispatcherServlet이 모든 요청을 가로채는 탓에 HTML/CSS/Javascript 같은 정적인 리소스를 응답하지 못하는 경우가 발생했다. 이를 해결 하기 위해 2 가지 방법을 고안하였다.<br/>

**정적인 자원에 대한 요청과 애플리케이션에 대한 요청 분리**

클라이언트의 요청을 2 가지로 분리하는 방법이다.
- `/apps`로 접근하면 DispatcherServlet이 담당
- `/resource`로 접근하면 DispatcherServlet이 담당 하지 않음

이 방식은 요청을 적절히 처리할 수 있었지만 코드가 지저분해지고 모든 요청에 대해서 해당 URL를 붙여주어야하므로 직관적인 설계가 될 수 없다.<br/>

**애플리케이션에 대한 요청을 탐색하고 없으면 정적 자원에 대한 요청으로 처리**

모든 요청을 컨트롤러에 두고 DispatcherServlet에서 해당 요청에 대한 컨트롤러를 찾을 수 없는 경우 2차적으로 설정된 자원 경로를 탐색하여 자원을 찾아내는 것이다. 이렇게 영역을 분리하여 효율적인 리소스 관리가 가능해지고 추후에 확장을 용이하게 해준다.<br/>

**DispatcherServlet을 web.xml로 등록하는 방법**
```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>

</web-app>

```

**DispatcherServlet을 자바 코드로 등록하는 방법**
```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register(AppConfig.class);

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic registration = servletContext.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```
### HandlerMapping
클라이언트의 요청을 처리할 수 있는 핸들러(컨트롤러)를 찾아 반환해주는 컴포넌트이다. 주요 구현체로는 
- RequestMappingHandlerMapping: 애노테이션 기반 핸들러인 `@RequestMapping`에서 사용
- BeanNameUrlHandlerMapping: 요청 URI와 동일한 이름을 가진 핸들러를 매핑한다.
- SimpleUrlHandlerMapping: Ant 스타일의 경로 매핑 방식을 사용하여 URI와 핸들러를 매핑한다.

### HandlerAdapter
핸들러 매핑이 반환하는 핸들러는 타입이 다르다. 따라서 DispatcherServlet은 핸들러를 직접 실행하지 않고 핸들러를 실행할 수 있는 어답터를 찾아 요청 처리를 위임한다.
- RequestMappingHandlerAdapter: 애노테이션 기반 핸들러인 `@RequestMapping`에서 사용
- HttpRequestHandlerAdapter: HttpRequestHandler를 처리한다. HttpServletRequest, HttpServletResponse를 사용하며 서블릿 처럼 동작하고 반환 값이 없다
- SimpleControllerHandlerAdapter: ModelAndView 객체를 반환하고, HttpServletRequest, HttpServletResponse를 사용하는 컨트롤러에 사용
### ViewResolver
뷰의 논리적인 이름(경로)와 실제 뷰를 매핑하는 역할을 한다. 
- BeanNameViewResolver: 빈 이름으로 뷰를 찾아서 반환한다
- InternalResourceViewResolver: JSP를 사용하여 뷰를 생성한다. prefix, suffix와 뷰 이름을 사용하여 실제 뷰를 매핑한다.

JSP는 `forward()`를 통해 렌더링 되지만 다른 뷰 템플릿을 `forward()` 호출 과정 없이 바로 렌더링된다.

## Spring MVC의 흐름

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/133201605-87be08f9-3cfc-414a-aa6e-ed36db93d71a.png width=600>
</p>

1. 클라이언트의 요청을 DispatcherServlet이 가로챈다
2. HanddlerMapping이 요청을 처리할 수 있는 핸들러를 조회하고 핸들러를 반환한다.
   1. RequestMappingHandlerMapping를 먼저 조회한다. 매핑 되지 않으면 넘어간다
   2. BeanNameUrlHandlerMapping를 조회한다
3. 핸들러를 실행할 수 있는 HandlerAdapter를 조회한다.
   1. HandlerAdapter의 `supports()`를 순서대로 호출한다
   2. 실행가능한 HandlerAdapter를 찾는다.
4. HandlerAdapter를 통해 핸들러를 실행하고 결과를 반환한다.
5. 반환 값에서 뷰의 이름을 View Resolver에게 전달하여 View를 찾는다
   1. BeanNameViewResolver을 통해 빈 이름으로 뷰를 찾는다.
   2. InternalResourceViewResolver을 통해 JSP를 처리할 수 있는 뷰를 찾는다
6. JSP의 경우 `view.render()`가 호출되고 JSP가 렌더링된다
7. 클라이언트에게 결과를 전달한다

DispatcherServlet의 핵심 메소드인 `doDispatch()`를 살펴본다.

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    ...

    ModelAndView mv = null;

    ...

    processedRequest = checkMultipart(request);
    multipartRequestParsed = (processedRequest != request);

    // 요청을 처리할 핸들러를 찾는다.
    mappedHandler = getHandler(processedRequest);
    if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return;
    }

    // 핸들러를 실행할 핸들러 어답터를 찾는다
    HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

    ...
    
    // 핸들러 어답터가 핸들러를 실행한다
    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

    // 요청 결과를 렌더링
    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    
    ...
}

private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
        @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
        @Nullable Exception exception) throws Exception {

    ...
    // 렌더링
    render(mv, request, response);
    ...
}

protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
    View view;
    String viewName = mv.getViewName(); // ModelAndView에서 뷰 이름을 가져온다

    ...

    // 뷰 리졸버를 통해 뷰 객체를 가져온다
    view = resolveViewName(viewName, mv.getModelInternal(), locale, request);

    ...
    // 렌더링한다.
    view.render(mv.getModelInternal(), request, response);
    
    ...
}

protected View resolveViewName(String viewName, @Nullable Map<String, Object> model,
        Locale locale, HttpServletRequest request) throws Exception {

    if (this.viewResolvers != null) {
        // 뷰 객체를 반환할 수 있는 뷰 리졸버를 찾는다.
        for (ViewResolver viewResolver : this.viewResolvers) {
            View view = viewResolver.resolveViewName(viewName, locale);
            if (view != null) {
                return view;
            }
        }
    }
    return null;
}
```
---

**참고**
- https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-view
- https://m.blog.naver.com/jhc9639/220967034588
- https://mangkyu.tistory.com/18
- https://javacan.tistory.com/tag/BeanNameUrlHandlerMapping
- https://springsource.tistory.com/2
- https://nanoson.tistory.com/33