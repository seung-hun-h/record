# 스프링 MVC 기본 기능
## 매핑 정보
### @Controller
- 반환 값이 String이면 뷰 이름으로 인식된다.

### @RestController
- 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력한다

## 로그
### 올바른 로그 사용법
- `log.debug("data = " + data)`
  - 로그 출력 레벨을 info로 설정해도 해당 코드에 있는 "data = " + data 가 살제 실행되어 버린다
  - 결과적으로 문자 더하기 연산이 발생한다

- `log.debug("data = {}", data)`
  - 로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다
  - 따라서 앞과 같은 의미없는 연산이 발생하지 않는다.

### 로그 사용시 장점
- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다
- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절할 수 있다
- 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등 로그를 별도의 위치에 남길 수 있다
  - 특히 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다
- 성능도 일반 System.out 보다 좋다

## HTTP 요청
### 기본, 헤더 조회
```java
@RestController
public class RequestHeaderController{
    @RequestMapping("/headers")
    public String headers(
        HttpServletRequest request,
        HttpServletResponse response,
        HttpMethod httpMethod,
        Locale locale,
        @RequestHeader MultiValueMap<String, String> headerMap,
        @RequestHeader("host") String host,
        @CoolieValue(value = "myCookie", required false) String cookie
    ) {
        ...
    }
}
```

### 쿼리 파라미터, HTML Form
### @RequestParam
```java
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(
    @RequestParam("username") String username,
    @RequestParam("age") int memberAge
) {
    ...
}
```

- @RequestParam은 파라미터 이름으로 바인딩 한다
  - 변수 이름이 파라미터 이름과 동일하면 name은 생략 가능하다
  - String, int, Integer 등 단순 타입이면 @RequestParam도 생략 가능하다
  - defaultValue를 설정해서 초기값도 설정할 수 있다 
    - 빈 문자의 경우에도 설정한 기본 값이 적용된다

### @ModelAttribute
```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
    ...
}
```

- 요청 파라미터를 객체로 받을 때 사용한다
- 스프링 MVC는 @ModelAttribute가 있으면 다음을 실행한다
  - HelloData 객체를 생성한다
  - 요청 파라미터 이름으로 HelloData 객체의 프로퍼티를 찾는다
  - 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력한다
    - 파라미터 이름이 username이면 setUsername() 메서드를 찾아 호출한다
- `age=abc` 처럼 숫자가 들어가야할 곳에 문자를 넣으면 BindException이 발생한다

- @ModelAttribute는 생략할 수 있다

### 단순 텍스트
- InputStream, OutputStream
  - InputStream: HTTP 요청 메시지 바디의 내용을 직접 조회
  - OutputStream: HTTP 응답 메시지의 바디에 직접 결과 출력

```java
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
    String messageBody = StreamUtils.copyToString(inputStream, StandardChasets.UTF_8);
    log.info*("messageBody = {}", messageBody);
    responseWriter.write("ok")
}
```

- HttpEntity
  - HTTP header, body 정보를 편리하게 조회할 수 있다
  - 바디 정보를 조회하는 것이지, 요청 파라미터를 조회하는 것과는 관련 없다
  - 응답에서도 사용 가능하다
    - 바디와 헤더 정보 포함 가능하다
  - HttpEntity를 상속받는 RequestEntity, ResponseEntity가 있다
```java
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
    String messageBody = httpEntity.getBody();
    log.info("messageBody = {}", messageBody);
    return new HttpEntity<>("ok");
}
```

- @RequestBody
  - 편리하게 HTTP 바디 메시지 정보를 조회할 수 있다
  - 헤더 정보가 필요하면 @RequestHeader, HttpEntity를 사용하면 된다

```java
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody) {
    log.info("messageBody={}", messageBody);
    return "ok";
}
```

- @RequestBody, HttpEntity를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다
- 컨버터는 문자 뿐 아니라 JSON 객체도 변환해준다
- @RequestBody는 생략이 불가능하다. 생략한다면 @ModelAttribute가 적용될 것이다.

## HTTP 응답
- 스프링(서버)에서 응답 데이터를 만드는 방법은 크게 3가지이다
- 정적 리소스
  - 웹 브라우저에 정적인 HTML, CSS, JS를 제공할 때는 정적 리소스를 사용한다
- 뷰 템플릿 사용
  - 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용한다
- HTTP 메시지 사용
  - HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다

### 정적 리소스, 뷰 템플릿
- 스프링 부트는 클래스패스에 다음 디렉토리에 있는 정적 리소스를 제공한다
  - /static, /public, /resources, /META-INF/resources
- src/main/resources는 리소스를 보관하는 곳이고, 또 클래스패스의 시작 경로이다
  - 따라서 다음 디렉토리에 리소스를 넣어두면 스프링 부트가 정적 리소스로 서비스를 제공한다
  - 정적 리소스 경로: src/main/resources/static
  - 정적 리소스는 해당 파일을 변경 없이 그대로 서비스 하는 것이다

- 스프링 부트는 기본 뷰 템플릿 경로를 제공한다
  - 뷰 템플릿 경로: src/main/resources/templates
  - @ResponseBody가 없이 String을 반환하면 뷰 리졸버가 실행되어 뷰를 찾고 렌더링한다
  - @Controller를 사용하고 HTTP 메시지 바디를 처리하는 파라미터가 없으면 요청 URL을 참고해서 논리 뷰 이름으로 사용한다.
    - 권장하지 않는 방법이다

### HTTP API, 메시지 바디에 직접 입력
```java
@Controller
//@RestController
public class ResponseBodyController {
    @GetMapping("/response-body-string-v1")
    public void responseBodyV1(HttpServletResponse response) throws IOException {
        response.getWriter().write("ok");
    }
    /**
    * HttpEntity, ResponseEntity(Http Status 추가)
    * @return
    */
    @GetMapping("/response-body-string-v2")
    public ResponseEntity<String> responseBodyV2() {
        return new ResponseEntity<>("ok", HttpStatus.OK);
    }
    
    @ResponseBody
    @GetMapping("/response-body-string-v3")
    public String responseBodyV3() {
        return "ok";
    }
    
    @GetMapping("/response-body-json-v1")
    public ResponseEntity<HelloData> responseBodyJsonV1() {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        helloData.setAge(20);
        return new ResponseEntity<>(helloData, HttpStatus.OK);
    }
    
    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    @GetMapping("/response-body-json-v2")
    public HelloData responseBodyJsonV2() {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
           helloData.setAge(20);
          return helloData;
    } 
}
```
- responseBodyV1
  - 서블릿을 직접 다루듯이 처리
  - HttpServletResponse 객체를 통해서 HTTP 메시지 바디에 직접 데이터를 넣는다
- responseBodyV2
  - HttpEntity를 상속한 ResponseEntity를 반환한다
  - 응답 코드까지 전달할 수 있다
- responseBodyV3
  - @ResponseBody를 사용하면 view를 사용하지 않고, HTTP 메시지 컨버터를 통해서 HTTP 메시지를 직접 입력할 수 있다
  - ResponseEntity도 동일한 방식으로 동작한다
- responseBodyJsonV1
  - ResponseEntity를 반환한다
  - HTTP 메시지 컨버터를 통해서 JSON 형식으로 변환되어 반환된다
- responseBodyJsonV2
  - ResponseEntity는 HTTP 응답 코드를 설정할 수 있는데, @ResponseBody를 사용하면 이런 것을 설정하기 까다롭다
  - @ResponseStatus를 사용하면 응답 코드도 설정할 수 있다
- @RestController
  - 해당 컨트롤러에 모두 @ResponseBody가 적용되는 효과가 있다
  - 뷰 템플릿을 사용하지 않고 HTTP 메시지 바디에 직접 데이터를 입력한다

## HTTP 메시지 컨버터
### @ResponseBody 사용 원리
![Screen Shot 2022-07-22 at 10 49 30 AM](https://user-images.githubusercontent.com/60502370/180344266-d422678d-b941-47bd-8186-6ca2f4cd6c68.png)

- HTTP의 바디에 문자 내용을 직접 반환
- viewResolver 대신에 HttpMessageConverter가 동작
- 기본 문자처리: StringHttpMessageConverter
- 기본 객체처리: MappingJackson2HttpMessageConverter
- byte처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있다
- 응답의 경우 클라이언트의 HTTP Accept 헤더와 서버 컨트롤러 반환 타입 정보 둘을 조합해서 HttpMessageConveter가 선택된다

- 스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다
  - HTTP 요청: @RequestBody, HttpEntity(RequestEntity)
  - HTTP 응답: @ResponseBody, HttpEntity(ResponseEntity)

### HttpMessageConverter
```java
package org.springframework.http.converter;
public interface HttpMessageConverter<T> {
    boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
    boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);
    List<MediaType> getSupportedMediaTypes();
    T read(Class<? extends T> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException;
    void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException;
}
```
- canRead, canWrite: 메시지 컨버터가 해당 클래스, 미디어 타입을 지원하는지 체크
- read, write: 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

- 스프링 부트 기본 메시지 컨버터
0. ByteArrayHttpMessageConverter
   - byte[] 데이터를 처리한다
   - 클래스 타입: byte[], 미디어 타입: `*/*`
   - 요청 예) @RequestBody byte[] data
   - 응답 예) @ResponseBody return byte[], 쓰기 미디어타입 application/octet-stream
1. StringHttpMessageConverter
   - String 데이터를 처리한다
   - 클래스 타입: String, 미디어 타입: `*/*`
   - 요청 예) @RequestBody String data
   - 응답 예) @ResponseBody return "ok", 쓰기 미디어타입 text/plain
2. MappingJackson2HttpMessageConverter
   - 클래스 타입: 객체 또는 HashMap, 미디어 타입: `application/json` 관련
   - 요청 예) @RequestBody HelloData data
   - 응답 예) @ResponseBody return helloData, 쓰기 미디어타입 application/json

### HTTP 요청 데이터 읽기
- HTTP 요청이 오고, 컨트롤러에서 @RequestBody, HttpEntity 파라미터를 사용한다
- 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead를 호출한다
  - 대상 클래스 타입을 지원하는가
  - HTTP 요청의 Content-Type 미디어 타입을 지원하는가
- canRead 조건을 만족하면 read를 호출해서 객체 생성하고 반환한다

### HTTP 응답 데이터 생성
- 컨트롤러에서 @ResponseBody, HttpEntity로 값이 반환된다
- 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 canWrite를 호출한다
  - 대상 클래스 타입을 지원하는가
  - HTTP 요청의 Accept 미디어 타입을 지원하는 가
- canWrite 조건을 만족하면 write를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다

## 요청 매핑 핸들러 어뎁터 구조
![Screen Shot 2022-07-22 at 11 03 34 AM](https://user-images.githubusercontent.com/60502370/180346810-de6b6cfc-e44b-4786-bd44-375510498e86.png)

- HttpMessageConverter는 위 그림에서 보이지 않는다
- @RequestMapping을 처리하는 RequestMappingHandlerAdapter를 확인하면 알 수 있다

### RequestMappingHandlerAdapter 동작 방식
![Screen Shot 2022-07-22 at 11 05 15 AM](https://user-images.githubusercontent.com/60502370/180346972-94846b2e-f2a6-4cfe-9699-557ed5c7b70c.png)

1. 컨트롤러의 파라미터, 애노테이션 정보를 기반으로 전달 데이터 생성
2. 핸들러 호출
3. 컨트롤러의 반환값 변환

### ArgumentResolver
- 애노테이션 기반 컨트롤러는 매우 다양한 파라미터를 사용할 수 있는데 이는 ArgumentResolver 덕분이다
- RequestMappingHandlerAdapter는 ArgumentResolver를 호출해서 컨트롤러가 필요로하는 다양한 파라미터의 값을 생성한다
  - 파라미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다
  - 스프링은 30개가 넘는 ArgumentResolver를 기본으로 제공한다

```java
public interface HandlerMethodArgumentResolver {
    boolean supportsParameter(MethodParameter parameter);

    @Nullable
    Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer, NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;
}
```

- ArgumentResolver의 supportsParameter를 호출해서 해당 파라미터를 지원하는지 체크한다
- 지원하는 경우  resolveArgument를 호출해서 실제 객체를 생성한다
- 생성된 객체는 컨트롤러로 전달된다

### ReturnValueHandler
- 응답 값을 변환하고 처리한다
- 컨트롤러에서 String으로 뷰 이름을 반환해도 동작하는 이유가 바로 ReturnValueHandler 덕분이다
- 스프링은 10개가 넘는 ReturnValueHandler를 제공한다

![image](https://user-images.githubusercontent.com/60502370/180347736-e696a6dd-b6bb-4cf1-91f7-5c8ce18a9139.png)

- ArgumentResolver와 ReturnValueHandler가 HttpMessageConverter를 호출한다