# MVC 프레임워크 만들기
## 프론트 컨트롤러 패턴
### 프론트 컨트롤러 패턴 도입 전
![Screen Shot 2022-07-20 at 9 42 38 AM](https://user-images.githubusercontent.com/60502370/179896468-2d3d0c1c-1781-434b-9ce3-d95d99aa7e3e.png)

### 프론트 컨트롤러 패턴 도입 후
![Screen Shot 2022-07-20 at 1 24 54 PM](https://user-images.githubusercontent.com/60502370/179896564-f06220de-48f7-4d37-b8c5-e8e18067f2e3.png)

### 프론트 컨트롤러 패턴의 특징
- 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받는다
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출한다
- 프론트 컨트로러에서 공통 처리가 가능하다
- 프론트 컨트롤러를 제외한 다른 컨트롤러에서는 서블릿을 사용하지 않아도 된다
- Spring MVC의 DispatcherServlet이 프론트 컨트롤러의 역할을 수행한다

### 프론트 컨트롤러 도입 - v1

![Screen Shot 2022-07-20 at 1 40 09 PM](https://user-images.githubusercontent.com/60502370/179898323-f5182edd-69cc-423d-8493-afe65c164a8b.png)

```java
public interface ControllerV1 {
    void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```
```java
public class MemberSaveControllerV1 implements ControllerV1 {
    private MemberRepository memberRepository = MemberRepository.getInstance();
    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        
        Member member = new Member(username, age);
        memberRepository.save(member);
        
        request.setAttribute("member", member);
        
        String viewPath = "/WEB-INF/views/save-result.jsp";
        
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

- 요청 URL에 따라 처리할 컨트롤러들은 `MemberSaveControllerV1` 처럼 ControllerV1 인터페이스를 구현한다

```java
@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {
    private Map<String, ControllerV1> controllerMap = new HashMap<>();

    public FrontControllerServletV1() {
        controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
        controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
    }
    
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("FrontControllerServletV1.service");
        
        String requestURI = request.getRequestURI();
        ControllerV1 controller = controllerMap.get(requestURI);
        
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND); 
            return; 
        }
        
        controller.process(request, response);
    }
}
```

- urlPatterns
  - `/front-controller/v1`를 포함한 하위 모든 요청은 해당 서블릿에서 받아드린다
- controllerMap
  - key: 매핑 URL
  - value: 호출될 컨트롤러

### View 분리 - V2

![Screen Shot 2022-07-20 at 1 40 34 PM](https://user-images.githubusercontent.com/60502370/179898373-ca75b69d-3c75-4d1c-8f10-b4f48eb420fb.png)

```java
String viewPath = "/WEB-INF/views/new-form.jsp";
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, respnose);
```
- 모든 컨트롤러에서 뷰로 이동하는 부분에 중복이 있다
- 이를 해결하기 위해서 별도로 뷰를 처리하는 객체를 만든다

```java
public class MyView {
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, respnose);
    }
}
```

```java
public interface ControllerV2 {
    MyView pprocess(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

```java
public class MemberSaveControllerV2 implements ControllerV2 {
    private MemberRepository memberRepository = MemberRepository.getInstance();
   
    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        
        Member member = new Member(username, age);
        memberRepository.save(member);
        
        request.setAttribute("member", member);
        
        String viewPath = "/WEB-INF/views/save-result.jsp";

        return new MyView(viewPath);
    }
}
```

```java
@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v2/*")
public class FrontControllerServletV2 extends HttpServlet {
    private Map<String, ControllerV1> controllerMap = new HashMap<>();

    public FrontControllerServletV1() {
        controllerMap.put("/front-controller/v2/members/new-form", new MemberFormControllerV2());
        controllerMap.put("/front-controller/v2/members/save", new MemberSaveControllerV2());
        controllerMap.put("/front-controller/v2/members", new MemberListControllerV2());
    }
    
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("FrontControllerServletV2.service");
        
        String requestURI = request.getRequestURI();
        ControllerV1 controller = controllerMap.get(requestURI);
        
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND); 
            return; 
        }
        
        MyView myView = controller.process(request, response);
        myView.render(request, response);
    }
}
```

### Model 추가 - V3
- 서블릿 종속성 제거
  - 컨트롤러 입장에서 HttpServletRequest, HttpServletResponse 객체가 반드시 필요한 것은 아니다
  - 요청 파라미터 정보는 자바의 Map으로 대신 넘기면 지금 구조에서 컨트롤러가 서블릿을 몰라도 동작할 수 있다
  - 그리고 HttpServletRequest를 사용하는 대신에 Model 객체를 별도로 만들어서 반환하면 된다
- 뷰 이름 중복 제거
  - 컨트롤러에서 뷰 이름이 중복된다
  - 컨트롤러는 뷰의 논리 이름을 반환하고, 실제 물리 위치의 이름은 프론트 컨트롤러에서 처리하도록 단순화 할 수 있다

<img width="564" alt="Screen Shot 2022-07-21 at 2 24 15 PM" src="https://user-images.githubusercontent.com/60502370/180136034-43bb88ff-4dcf-40ba-9c85-bc8e415166b4.png">

- ModelView
  - 서블릿 종속성을 제거하기 위해 Model을 직접 만들고, 추가로 View 이름까지 전달하는 객체를 만든다

```java
@Getter
@Setter
public class ModelView {
    private String viewName;
    private Map<String, Object> model = new HashMap<>();

    public ModelView(String viewName) {
        this.viewName = viewName;
    }
}
```

```java
public interface ControllerV3 {
    ModelView process(Map<String, String> paramMap);
}
```

- ControllerV3은 서블릿 기술을 전혀 사용하지 않는다
  - 구현이 단순해지고, 테스트 코드를 작성하기 쉽다

```java
public class MemberSaveControllerV3 implements ControllerV3 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelView mv = new ModelView("save-result");
        mv.getModel().put("member", member);
        return mv;
    }
}
```

```java
@WebServlet(name="frontControllerServletV3", urlPatterns = "/front-controller/v3")
public class FrontControllerServletV3 extends HttpServlet {
    private Map<String, ControllerV3> controllerMap = new HashMap<>();

    public FrontControllerServletV3() {
        controllerMap.put("/front-controller/v3/members/new-form", new MemberFormControllerV3());
        controllerMap.put("/front-controller/v3/members/save", new MemberSaveControllerV3());
        controllerMap.put("/front-controller/v3/members", new MemberListControllerV3());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String requestURI = request.getRequestURI();

        ControllerV3 controller = contollerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        String viewName = mv.getViewName();
        MyView view = viewResolver(viewName);
        view.render(mv.getModel(), request, resonse);
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();

        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));

        return paramMap;
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}
```

```java
public class MyView {
    ...
    public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        modelToRequestAttribute(model, request);
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, resopnse);
    }

    private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
        model.forEach((key, value) -> request.setAttribute(key, value));
    }
}
```

### 단순하고 실용적인 컨트롤러 - V4
- 좋은 프레임워크는 아키텍처도 중요하지만, 개발자가 사용하기 편리해야 한다

<img width="564" alt="Screen Shot 2022-07-21 at 2 46 46 PM" src="https://user-images.githubusercontent.com/60502370/180138690-08d07a32-5337-4504-afdf-7cdeeee8e39f.png">

- 기본적인 구조는 v3와 같지만 대신, 컨트롤러가 ModelView가 아닌 viewName만 반환한다

```java
public interface ControllerV4 {
    String process(Map<String, String> paramMap, Map<String, Object> model);
}
```

- model 객체는 파라미터로 전달되고, 뷰 이름만 반환한다

```java
public class MemberSaveControllerV4 implements ControllerV4 {
    private MemberRepository memberRepository = MemberRepository.getInstance();
    
    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));
       
        Member member = new Member(username, age);
        memberRepository.save(member);
       
        model.put("member", member);
       
        return "save-result";   
    } 
}
```

```java
@WebServlet(name="frontControllerServletV4", urlPatterns = "/front-controller/v4")
public class FrontControllerServletV4 extends HttpServlet {
    private Map<String, ControllerV4> controllerMap = new HashMap<>();

    public FrontControllerServletV4() {
        controllerMap.put("/front-controller/v4/members/new-form", new MemberFormControllerV4());
        controllerMap.put("/front-controller/v4/members/save", new MemberSaveControllerV4());
        controllerMap.put("/front-controller/v4/members", new MemberListControllerV4());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String requestURI = request.getRequestURI();

        ControllerV4 controller = contollerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>();

        String viewName = controller.process(paramMap, model);

        MyView view = viewResolver(viewName);
        view.render(mv.getModel(), request, resonse);
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();

        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));

        return paramMap;
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}
```

- 프레임워크나 공통 기능이 수고로워야 사용하는 개발자가 편리해진다

### 유연한 컨트롤러1 - V5
- 어떤 개발자는 ContollerV3 방식으로 개발하고 싶고, 다른 개발자는 ControllerV4 방식으로 개발하고 싶을 수 있다
- 어댑터 패턴
  - 어댑터 패턴을 사용하면 완전히 다른 두 컨트롤러를 사용할 수 있게된다

<img width="564" alt="Screen Shot 2022-07-21 at 2 59 52 PM" src="https://user-images.githubusercontent.com/60502370/180140342-3a396045-2f0a-4dbb-9ec5-84d51c655f26.png">

- 핸들러 어댑터
  - 중간에 어댑터 역할을 하는 어댑터가 추가되었는데 이름이 핸들러 어댑터이다
  - 어댑터 덕분에 다양한 종류의 컨트롤러를 호출할 수 있다
- 핸들러
  - 컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경했다
  - 컨트롤러 뿐아니라 어떤것이든 해당하는 종류의 어댑터만 있으면 다 처리할 수 있다

```java
public interface MyHandlerAdapter {
    boolean supports(Object handler);

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```

- supports
  - 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드다
- handle
  - 어댑터는 실체 컨트롤러를 호출하고, 그 결과로 ModelView를 반환해야 한다
  - 실제 컨트롤러가 ModelView를 반환하지 못하면, 어댑터가 ModelView를 직접 생성해서라도 반환해야 한다
  - 이제 어댑터를 통해서 실제 컨트롤러가 호출된다

```java
public class ControllerV3HandlerAdapter implements MyHandlerAdapter {
    
    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV3);
    }
    
    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) {

        ControllerV3 controller = (ControllerV3) handler;
        Map<String, String> paramMap = createParamMap(request);
    
        ModelView mv = controller.process(paramMap);
        return mv; 
    }
    
    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```

```java
public class ControllerV4HandlerAdapter implements MyHandlerAdapter {
    
    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV4);
    }
    
    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) {

        ControllerV4 controller = (ControllerV4) handler;
        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>();
    
        String viewName = controller.process(paramMap, model);

        ModelView mv = new ModelView(viewName);
        mv.setModel(model);
        
        return mv; 
    }
    
    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```

```java
@WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-
controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {
    private final Map<String, Object> handlerMappingMap = new HashMap<>();
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();
    
    public FrontControllerServletV5() {
        initHandlerMappingMap();
        initHandlerAdapters();
    }

    private void initHandlerMappingMap() {
        handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
         handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());
    }
    
    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdapter());
    }
    
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Object handler = getHandler(request);
        
        if (handler == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return; 
        }
        
        MyHandlerAdapter adapter = getHandlerAdapter(handler);
        ModelView mv = adapter.handle(request, response, handler);
        
        MyView view = viewResolver(mv.getViewName());
        view.render(mv.getModel(), request, response);
    }

    private Object getHandler(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        return handlerMappingMap.get(requestURI);
    }
    
    private MyHandlerAdapter getHandlerAdapter(Object handler) {
        for (MyHandlerAdapter adapter : handlerAdapters) {
            if (adapter.supports(handler)) {
                return adapter;
            } 
        }

        throw new IllegalArgumentException("handler adapter를 찾을 수 없습니다. handler=" + handler);
    }

    private MyView viewResolver(String viewName) {
          return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    } 
}
```