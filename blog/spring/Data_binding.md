오늘은 HTTP 요청 파라미터 조회와 HTTP 메시지 바디를 직접 조회하는 방법에 대해서 알아본다.

---

## @RequestParam
---

@RequestParam는 HTTP 요청 파라미터를 바인딩 하기 위해 사용된다. `@RequestParam(name = xxx)`과 같이 파라미터 이름을 지정하면 해당 파라미터와 데이터가 바인딩된다.<br/>

파라미터의 이름과 바인딩할 인자의 이름이 동일한 경우 name을 생략할 수 있다. 또한 이름이 동일하고 타입이 String, int, Integer와 같은 단순한 타입이라면 `@RequestParam`까지 생략 가능하다.

```java

...

@GetMapping("/posts")
public ResponseEntity<Post> requestParam(@RequestParam("postId") Long postId) {
    Post post = postService.findById(postId);
    return ResponseEntity.ok(post);
}

// 파라미터 이름과 인자의 이름이 동일
@GetMapping("/posts")
public ResponseEntity<Post> requestParam(@RequestParam Long postId) {
    Post post = postService.findById(postId);
    return ResponseEntity.ok(post);
}

// 인자의 타입이 Long으로 단순한 타입
@GetMapping("/posts")
public ResponseEntity<Post> requestParam(Long postId) {
    Post post = postService.findById(postId);
    return ResponseEntity.ok(post);
}
```

파라미터의 값이 반드시 필요한 것이 아니라면 `required=false`로 속성을 지정할 수 있다. 기본 값은 true 이지만,  **만약 `@RequestParam`을 생략했다면 Spring MVC에서 자동으로 `required=false`로 설정한다.**<br/>

파라미터를 필수로 설정했을 경우 파라미터가 전달되지 않으면 400 Error가 발생한다. 그리고 파라미터를 전달하지 않을 경우를 대비해 `defaultValue` 속성을 통해 **기본값을 설정**할 수 있다. **`defaultValue`는 빈 문자를 전달 받을 때도 동작한다.**


```java

...

// 파라미터 필수
@GetMapping("/posts")
public ResponseEntity<Post> requestParam(@RequestParam(required = true) Long postId) {
    Post post = postService.findById(postId);
    return ResponseEntity.ok(post);
}

// 파라미터 옵셔널
@GetMapping("/posts")
public ResponseEntity<Post> requestParam(@RequestParam(required = false) Long postId) {
    Post post = postService.findById(postId);
    return ResponseEntity.ok(post);
}

// 파라미터 기본 값 설정
@GetMapping("/posts")
public ResponseEntity<Post> requestParam(@RequestParam(defaultValue = "1") Long postId) {
    Post post = postService.findById(postId);
    return ResponseEntity.ok(post);
}
```

파라미터를 Map으로도 조회할 수 있다. 파라미터의 값이 1개가 확실하다면 `Map`을 사용하고 그렇지 않다면 `MultiValueMap`을 사용한다


```java
@GetMapping("/posts")
public ResponseEntity<Post> requestParam(@RequestParam Map<String, Object> map) {
    Post post = postService.findById(map.get("postId"));
    return ResponseEntity.ok(post);
}
```
### @RequestParam 정리

- HTTP 파라미터를 조회할 때 사용한다.
- name, required, defaultValue 속성을 지정할 수 있다
- 파라미터와 변수의 이름이 같으면 name 속성 생략 가능하다
- 이름이 같고 단순한 값(Long 등)이면 어노테이션 자체를 생략 가능하다
- required 속성을 통해 파라미터의 필수 여부를 설정한다
- defaultValue는 기본 값을 설정한다
  - 빈 문자열도 기본 값으로 대체된다
  

## @ModelAttribute

---
@ModelAttribute는 클라리언트가 전송하는 form-data나 HTTP 요청 파라미터들을 **setter 메소드를 통해 객체에 바인딩**한다.

```java

public class PostDto {
    String title;
    String content;

    public void setTitle(String title) {
        this.title = title;
    }

    public void setContent(String content) {
        this.content = content;
    }

    ...
}

@PostMapping("/posts")
public ResponseEntity<Post> requestParam(@ModelAttribute PostDto dto) {
    Post post = postService.save(dto)
    return ResponseEntity.ok(post);
}

```

Spring MVC는 `@ModelAttribute`가 있을 경우
1. PostDto 객체를 생성한다
2. 요청 파라미터의 이름으로 PostDto 객체의 프로퍼티를 찾는다
3. 해당 프로퍼티의 setter를 호출하여 파라미터의 값을 입력한다

참고로 파라미터의 이름이 title이면 `setTitle()` 메서드를 찾아서 호출한다.

`@ModelAttribute` 또한 생략이 가능하다. 스프링은 어노테이션 생략시 다음과 같은 규칙을 적용한다
1. 단순한 값인 경우 -> @RequestParam
2. 나머지 -> @ModelAttribute

지금까지 살펴본 두 어노테이션은 HTTP 요청 파라미터를 바인딩하는 방법이다. HTTP 요청 파라미터는 `/url?key=value`와 같이 url에 노출되는 경우가 있고, form-data로 전달될 경우 `key=value`가 HTTP 메시지 바디로 전달될 수 있는데 두 경우 모두 요청파라미터로 인정된다.

바인딩 하고자는 데이터의 타입이 맞지 않을 때 (int 타입에 "안녕" 처럼) **BindException**이 발생한다.

## @RequestBody
---
`@RequestBody`는 클라이언트가 전송하는 HTTP 메시지 바디의 정보를 편리하게 조회할 수 있도록 도와준다. 이렇게 메시지 바디를 조회하는 기능은 위에서 살펴본 두 가지 기능과는 전혀 관련이 없다. <br/>

`@RequestBody`는 String이나 HttpEntity에 사용할 수 있다. 다만 String을 사용할 경우 ObjectMapper를 통해 다시 객체로 매핑해야하는 번거로움이 있으므로 일반적으로는 `@RequestBody`, `HttpEntity`를 사용한다.

```java
...
// String
public ResponseEntity<Post> requestBody(@RequestBody String messageBody) {
    PostDto postDto = objectMapper.readValue(messageBody, PostDto.class);

    Post post = postService.save(postDto);

    return ResponseEntity.ok(post);
}

// HttpEntity
public ResponseEntity<Post> requestBody(@RequestBody PostDto postDto) {
    Post post = postService.save(postDto);

    return ResponseEntity.ok(post);
}
```

`@RequestBody`, `HttpEntity`를 사용하면 스프링은 HttpMessageConverter 중 하나인 MappingJackson2HttpMessageConverter를 통해 변환된다. 메시지를 변환하는 과정에서 **객체의 기본 생성자를 통해 객체를 생성**하고, Reflection을 사용해 값을 할당한다. 이러한 이유로 **setter가 필요 없다.**<br/>

따라서 `@RequestBody`에 사용할 HttpEntity는 기본 생성자가 존재해야하며 private access modifier는 사용할 수 없다.<br/>

주의해야 할 점은 Http 요청 시에 content-type이 appllication/json인지 꼭 확인해야한다. 그래야 JSON을 처리할 수 있는 컨버터가 실행된다.<br/>

**그리고 @RequestBody는 생략할 수 없다. 만약 실수로 생략했다면 @ModelAttribute로 지정될 가능성이 높기 때문에 setter의 부재로 인한 에러가 발생할 수 있다.**

