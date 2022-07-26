# API 예외 처리
- API는 각 오류 상황에 맞는 오류 응답 스펙을 정하고 JSON으로 데이터를 내려주어야한다

```java
@Slf4j
@RestController
public class ApiExceptionController {
    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {
        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }

        return new MemberDtO(id, "hello " + id);
    }
    
    @Data
    @AllArgsConstructor
    static class MemberDto {
        private String memberId;
        private String name;
    }
}
```

- `id`를 `ex`로 요청하면 오류 페이지 HTML이 반환된다
- 이것은 기대하는 바가 아니다. 항상 JSON이 반환되기를 기대한다
- 문제를 해결하기 위해서는 오류 페이지 컨트로러도 JSON 응답을 할 수 있도록 수정해야 한다

```java
@RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
public ResponseEntity<Map<String, Object>> errorPage500Api(HttpServletRequest request, HttpServletReseponse response) {
    log.info("API errorPage 500");
    
    Map<String, Object> result = new HashMap<>();
    Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);

    result.put("status", request.getAttribute(ERROR_STATUS_CODE));
    result.put("message", ex.getMessage());

    Integer statusCode = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
    
    return new ResponseEntity(result, HttpStatus.valueOf(statusCode));
}
```
- 클라이언트의 HTTP Request Header의 Accept 값이 `application/json`인 경우 해당 메서드가 호출된다

## 스프링 부트 기본 오류 처리
### BasicErrorController 코드
```java
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse
response) {}

@RequestMapping
public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
```

- `errorHtml()`: Accept값이 `text/html`인 경우 호출되어 view를 제공한다
- `error()`: 그 외 경우에 호출된다. `ResponseEntity`로 HTTP Body에 JSON 데이터를 반환한다

### HTML 페이지 VS API 오류
- 스프링 부트가 제공하는 `BasicErrorController`는 HTML 페이지를 제공하는 경우 매우 편리하다
- 하지만 API 오류는 다른 이야기이다
  - API마다, 각각의 컨트롤러나 예외마다 서로 다른 응답 결과를 출력해야 할 수도 있다

## HandlerExceptionResolver 사용
### 목표
- 예외가 발생해서 서블릿을 넘어 WAS까지 예외가 전달되면 HTTP 상태코드가 500으로 처리된다
- 발생하는 예외에 따라서 400, 404 등등 다른 상태코드도 처리하고 싶다
- 오류 메시지, 형식 등을 API 마다 다르게 처리하고 싶다

### 상태 코드 변환

```java
@Slf4j
@RestController
public class ApiExceptionController {
    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {
        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }

        if (id.equals("bad")) {
            throw new RuntimeException("잘못된 입력 값");
        }

        return new MemberDtO(id, "hello " + id);
    }
    
    @Data
    @AllArgsConstructor
    static class MemberDto {
        private String memberId;
        private String name;
    }
}
```

- `/api/members/bad`를 호출하면 `IllegalArgumentException`이 발생한다
```json
{
    "status": 500,
    "error": "Internal Server Error",
    "exception": "java.lang.IllegalArgumentException",
    "path": "/api/members/bad"
}
```

### HandlerExceptionResolver
- 컨트롤러 밖으로 던져진 예외를 해결하고, 동작 방식을 변경하고 싶으면 `HandlerExceptionResolver`를 사용하면 된다

- ExceptionResolver 사용전

![image](https://user-images.githubusercontent.com/60502370/180935947-fd94b596-32b9-4f67-b60a-92431e13bb53.png)

- ExceptionResolver 사용후

![image](https://user-images.githubusercontent.com/60502370/180936029-d8d8f936-0fa0-4d43-b87c-c77c603365be.png)

- `HandlerExceptionResolver` 인터페이스

```java
public interface HandlerExceptionResolver {
    ModelAndView resolveException(
    HttpServletRequest request, HttpServletResponse response,
    Object handler, Exception ex);
}
```

- `HandlerExceptionResolver` 구현

```java
@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        try {
            if (ex instanceof IllegalArgumentException) {
                log.info("IllegalArgumentException resolver to 400");
                response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());
                return new ModelAndView();
            }
        } catch (IOException e) {
            log.error("resolver ex", e);
        }

        return null;
    }
}
```

- `ExceptionResolver`가 `ModelAndView`를 반환하는 이유는 마치 try-catch를 하듯이, `Exception`을 처리해서 정상 흐름으로 변경하는 것이 목적이다.
- `IllegalArgumentException`이 발생하면 `response.sendError(400)`을 호출해서 HTTP 상태코드를 저장하고 빈 `ModelAndView`를 반환한다

- 반환 값에 따른 동작 방식: `HandlerExceptionResolver`의 반환 값에 따른 `DispatcherServlet`의 동작 방식
  - 빈 `ModelAndView`: 뷰를 렌더링하지 않고 정상 흐름으로 서블릿이 리턴된다
  - `ModelAndView` 지정: View, Model 등의 정보를 지정해서 반환하면 뷰를 렌더링한다
  - `null`: 다음 `ExceptionResolver`를 찾아서 실행한다. 처리할 수 있는 `ExceptionResolver`가 없으면 서블릿 밖으로 예외를 던진다

### HandlerExceptionResolver 활용
- 예외 상태 코드 변환
  - 예외를 `respnose.sendError()` 호출로 변경해서 서블릿에서 상태 코드에 따른 오류를 처리하도록 위임
  - 이후 WAS는 서블릿 오류 페이지를 찾아서 내부 호출
- 뷰 템플릿 처리
  - `ModelAndView`에 값을 채워서 예외에 따른 새로운 오류 화면 뷰를 렌더링
- API 응답 처리
  - `response.getWriter().println("hello")`처럼 HTTP 응답 바디에 직접 데이터를 넣음
  - JSON으로 응답하면 API 응답처리 가능

### HandlerExceptionResolver 등록
```java
@Override
public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver>
resolvers) {
    resolvers.add(new MyHandlerExceptionResolver());
}
```

- `ExceptionResolver`는 컨트롤러에 예외가 발생해도 처리해버린다
  - 서블릿까지 예외가 전달되지 않고 스프링 MVC에서 예외 처리가 끝난다
  - 결과적으로 WAS 입장에서는 정상 처리가 된 것이다
- 하지만 예외마다 `ExceptionResolver`를 구현하여 등록하는 것은 번거롭다

## 스프링이 제공하는 ExceptionResolver1
- 스프링 부트가 기본으로 제공하는 `ExceptionResolver`는 다음과 같다
  - `ExceptionHandlerExceptionResolver`
    - `@ExceptionHandler`을 처리한다
    - API 예외 처리는 대부분 이 기능으로 해결한다
  - `ResponseStatusExceptionResolver`
    - HTTP 상태 코드를 지정해준다
    - `@ResponseStatus(value = HttpStatus.NOT_FOUND)`
  - `DefaultHandlerExceptionResolver`
    - 스프링 내부 기본 예외를 처리한다

### ResponseStatusExceptionResolver
- 예외에 따라서 HTTP 상태 코드를 지정해주는 역할을 한다
- 다음 두 가지 경우를 처리한다
  - `@ResponseStauts`가 달려있는 예외
  - `ResponseStautsException`예외

```java
@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "잘못된 요청 오류")
public class BadReqeustException extends RuntimeException {
}
```

- `BadRequestException`이 컨트롤러 밖으로 넘어가면 `ResponseStatusExceptionResolver` 예외가 해당 애노테이션을 확인해서 오류 코드를 `HttpStatus.BAD_REQUEST`로 변경하고 메시지도 담는다
- 코드를 확인해보면 결국 `response.sendError()`를 호출하는 것을 알 수 있다
  - WAS에서 다시 오류 페이지를 내부 요청한다

### ResponseStatusException
- `@ResponseStauts`는 개발자가 직접 변경할 수 없는 예외에는 적용할 수 없다
  - 애노테이션을 직접 넣어야하는데 코드를 수정할 수 없는 경우가 있다
- 추가로 애노테이션을 사용하기 때문에 조건에 따라 동적으로 변경하는 것도 어렵다
- 이때는 `ResponseStatusException`을 사용하면 된다
  
```java
@GetMapping("/api/response-status-ex2")
public String responseStatusEx2() {
    throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.bad", new IllegalArgumentException());
}
```

## 스프링이 제공하는 ExceptionResolver2
### DefaultHandlerExceptionResolver
- 스프링 내부에서 발생하는 스프링 예외를 해결한다
  - 파라미터 바인딩 시점에 타입이 맞지 않으면 내부에서 `TypeMismatchException`이 발생한다
  - 이 경우 예외가 발생헀기 때문에 그냥 두면 서블릿 컨테이너까지 오류가 올라가고 결과적으로 500 오류가 발생한다
  - 하지만 파라미터 바인딩은 대부분 클라이언트가 HTTP 요청 정보를 잘못 호출해서 발생하는 문제이다
  - HTTP에서는 이런경우 400코드를 사용하도록 되어 있다
  - `DefaultHandlerExceptionResolver`는 이것을 500이 아닌 400으로 변경한다

- `DefaultHandlerExceptionResolver.handleTypeMismatch`를 보면 `response.sendError(HttpServletResponse.SC_BAD_REQUEST)`를 확인할 수 있다
  - WAS에서 다시 오류 페이지를 내부 요청한다


## @ExceptionHandler
- API는 각 시스템마다 응답의 모양도 다르고 스펙도 모두 다르다
- 따라서 지금까지의 `BasicErrorController`를 사용하거나 `HandlerExceptionResolver`를 직접 구현하는 방식으로는 API 예외를 처리하기 쉽지않다

### API 예외를 처리하기 어려운 점
- `HandlerExceptionResolver`는 `ModelAndView`를 반환하지만 API 응답에는 필요하지 않다
- API 응답을 위해서는 `HttpServletResponse` 바디에 넣어주었다. 이것은 매우 불편하다
- 특정 컨트롤러에서 발생한 예외를 별도로 처리하기 어렵다

### @ExceptionHandler
- 스프링은 API 예외 처리 문제를 해결하기 위해 `@ExceptionHandler`라는 애노테이션을 사용하는 편리한 기능을 제공한다
- 이것이 `ExceptionHandlerExceptionResolver`이다. 실무에서는 대부분 이 방식을 사용한다

```java
@Data
@AllArgsConstructor
public class ErrorResult {
    private String code;
    private String message;
}
```

```java
@Slf4j
@RestController
public class ApiExceptionV2Controller {
    
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandle(IllegalArgumentException e) {
        log.error("[exceptionHandle] ex", e);
        return new ErrorResult("BAD", e.getMessage());
    }
    
    @ExceptionHandler
    public ResponseEntity<ErrorResult> userExHandle(UserException e) {
        log.error("[exceptionHandle] ex", e);
        ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
        return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
    }
    
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandle(Exception e) {
        log.error("[exceptionHandle] ex", e); return new ErrorResult("EX", "내부 오류");
    }
    // ...
}
```

- `@ExceptionHandler`을 선언하고 처리하고 싶은 예외를 지정한다
  - 자식 예외까지 모두 처리된다
- 항상 자세한 것이 우선한다
  - 부모보다 자식 예외 처리가 우선한다
- 다양한 예외를 한 번에 처리할 수 있다
- 예외를 생략할 수 있다
  - 예외를 생략하면 파라미터의 예외가 지정된다

### 실행 흐름
- 컨트롤러를 호출한 결과 `IllegalArgumentException`예외가 컨트롤러 밖으로 던져진다
- 예외가 발생했으므로 `ExceptionResolver`가 작동한다. 가장 우선순위가 높은 `ExceptionHandlerExceptionResolver`가 실행된다
- `ExceptionHandlerExceptionResolver`는 해당 컨트롤러에 `IllegalArgumentException`을 처리할 수 있는 `@ExceptionHandler`가 있는지 확인한다
- 해당 핸들러가 실행된다. `@RestController`이므로 HTTP 컨버터가 사용되고 응답이 JSON으로 반환된다
- `@ResponseStauts(HttpStatus.BAD_REQUEST)` 이므로 상태 코드가 400으로 응답된다

## @ControllerAdvice
- `@ExceptionHandler`를 사용하면 예외 처리를 깔끔하게 할 수 있다
- 하지만 정속 코드와 예외 처리 코드가 하나의 컨트롤러에 섞여있다
- `@ControllerAdvice` 혹은 `@RestControllerAdvice`을 사용하면 코드를 분리할 수 있다

```java
@Slf4j
@RestControllerAdvice
public class ExControllerAdvice {
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandle(IllegalArgumentException e) {
        log.error("[exceptionHandle] ex", e);
        return new ErrorResult("BAD", e.getMessage());
    }

    @ExceptionHandler
    public ResponseEntity<ErrorResult> userExHandle(UserException e) {
        log.error("[exceptionHandle] ex", e);
        ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
        return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
    }
    
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandle(Exception e) {
        log.error("[exceptionHandle] ex", e);
        return new ErrorResult("EX", "내부 오류"); 
    }
```

### @ControllerAdvice
- 대상으로 지정한 여러 컨트롤러에 `@ExceptionHandler`, `@InjectBinder` 기능을 부여해주는 역할을 한다
- 대상을 지정하지 않으면 모든 컨트롤러에 적용된다
- `@ControllerAdvice` 와 `@RestControllerAdvice`의 차이는 `@ResponseBody`의 여부이다

### 대상 컨트롤러 지정 방법
```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class,
AbstractController.class})
public class ExampleAdvice3 {}
```

