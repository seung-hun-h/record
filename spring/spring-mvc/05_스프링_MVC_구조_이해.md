# 스프링 MVC 구조의 이해
## 스프링 MVC 구조

<img width="564" alt="Screen Shot 2022-07-21 at 3 39 11 PM" src="https://user-images.githubusercontent.com/60502370/180146347-0f63eba2-3514-44b8-b469-9efdfee04d22.png">

### DispatcherServlet 구조 살펴보기
- 스프링 MVC는 프론트 컨트롤러 패턴으로 구현되어있다
- 스프링 MVC의 프론트 컨트롤러가 DispatcherServlet이다
- DispatcherServlet 서블릿 등록
  - DispatcherServlet도 부모 클래스에서 HttpServlet을 상속 받아서 사용하고, 서블릿으로 동작한다
    - DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
  - 스프링 부트는 DispatcherServlet 서블릿을 자동으로 등록하면서 모든 경로에 매핑한다
    - 더 자세한 경로가 우선 순위가 높다. 그래서 기존에 등록한 서블릿도 함께 동작한다
- 서블릿이 호출되면 HttpServlet이 제공하는 service()가 호출된다

### 동작 순서
<img width="564" alt="Screen Shot 2022-07-21 at 3 47 37 PM" src="https://user-images.githubusercontent.com/60502370/180148081-8adcd228-b362-4441-927b-a1c4993dba9e.png">

1. 핸들러 조회: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러를 조회한다
2. 핸들러 어댑터 조회: 핸들러를 실행할 수 있는 어댑터를 조회한다
3. 핸들러 어댑터 실행: 핸들러 어댑터를 실행한다
4. 핸들러 실행: 핸들러 어댑터가 핸들러를 실행한다
5. ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다
6. viewResolver 호출: 뷰 리졸버를 찾고 실행한다
   - JSP 경우 InternalResourceViewResolver가 자동 등록되고 사용된다
7. View 반환: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다
   - JSP의 경우 InternalResourceView(JstView)를 반환하는데, 내부에 forward() 로직이 있다
8. 뷰 렌더링: 뷰를 통해서 뷰를 렌더링한다

### 정리
- 스프링 MVC의 강점은 DispatcherServlet의 코드 변경 없이, 원하는 기능을 변경하거나 확장할 수 있다는 점이다

## 핸들러 매핑과 핸들러 어댑터
### HandelrMapping
0. RequestMappingHandlerMapping: 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
1. BeanNameUrlHandlerMapping: 스프링의 빈 이름으로 핸들러를 찾는다

### HandlerAdapter
0. RequestMappingHandlerAdapter: 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
1. HttpRequestHandlerAdapter: HttpRequestHandler 처리
2. SimpleControllerHandlerAdapter: Controller 인터페이스(애노테이션 x, 과거에 사용)

## 뷰 리졸버
### ViewResolver
1. BeanNameViewResolver: 빈 이름으로 뷰를 찾아서 반환한다
2. InternalResourceViewResolver: JSP를 처리할 수 있는 뷰를 반환한다

## 스프링 MVC 시작하기
### @RequestMapping
- 스프링은 애노테이션을 활용한 매우 유연하고, 실용적인 컨트롤러를 만들었는데 이것이 바로 @RequestMapping이다

```java
@Controller
public class SpringMemberFormControllerV1 {

    @RequestMapping("/springmvc/v1/members/new-form")
    public ModelAndView process() {
        return new ModelAndView("new-form");
    }
}
```

- @Controller
  - 스프링이 자동으로 스프링 빈으로 등록한다
  - 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식한다
- @RequestMapping
  - 요청 정보를 매핑한다
  - 해당 URL이 호출되면 이 메서드가 호출된다
  - 애노테이션 기반으로 동작하기 때문에 메서드의 이름은 임의로 지으면 된다

- RequestMappingHandlerMapping은 스프링 빈 중에서 @RequestMapping 또는 @Controller가 클래스 레벨에 붙어있는 경우에 매핑 정보로 인식한다
  - 따라서 다음 코드도 동일하게 동작한다

```java
@Component
@RequestMapping
public class SpringMemberFormControllerV1 {

    @RequestMapping("/springmvc/v1/members/new-form")
    public ModelAndView process() {
        return new ModelAndView("new-form");
    }
}
```

### 실용적인 방식
```java
@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping("/new-form")
    public String newForm() {
        return "new-form";
    }
    ...
}
```

- @RequestMapping은 URL 뿐 아니라 HTTP Method도 함께 구분할 수 있다

```java
@RequestMapping(value = "/new-form", method = RequestMethod.GET)
```

- 하지만 이것을 @GetMapping, @PostMapping으로 사용하면더 편리하다