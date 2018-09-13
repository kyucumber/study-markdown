## Spring @ControllerAdvice 어노테이션

스프링에서 예외처리를 전역적으로 핸들링하기 위해 @ControllerAdvicde 어노테이션을 사용할 수 있다.

아래와 같이 전역적으로 Exception을 핸들링 할 클래스를 만들고 어떤 에러에 대해서 어떤 방법으로 처리해줄지에 대해 정의해서 사용할 수 있다.

```java
@ControllerAdvice
public class ControllerExceptionHandler {
	private static final Logger log = LoggerFactory.getLogger(ControllerExceptionHandler.class);
	
	@ExceptionHandler(NoLoginException.class)
    public String noLoginException(Exception e, Model model) {
        model.addAttribute("errorMessage", e.getMessage());
        return "user/login_failed";
    }
}
```

위는 **NoLoginException**이 발생한 경우에 해당 에러메세지를 표시하는 페이지를 리턴해주도록 구현한 코드다. ControllerAdvice를 사용해서 아래의 형태로 응답을 줄수도 있다.

```java
@ExceptionHandler(AlreadyIncludeSharedNoteBookException.class)
public ResponseEntity alreadyIncludeShardNoteBook(AlreadyIncludeSharedNoteBookException exception) {
    log.debug("AlreadyIncludeSharedNoteBookException is happened!");
    ErrorDetails errorDetails = new ErrorDetails(new Date(), exception.getMessage());
    return ResponseEntity.status(HttpStatus.FORBIDDEN).body(errorDetails);
}
```

근데 위같은 경우에는 위의 방법보다는 @RestControllerAdvice를 사용하는게 좋을 듯

> @RestControllerAdvice는 @ControllerAdvice` + `@ResponseBody의 역할을 한다.

```java
@RestControllerAdvice
public class RestExceptionHandler {
  @ResponseStatus(value = HttpStatus.UNAUTHORIZED)
  @ExceptionHandler(value = UnAuthorizedException.class)
  public Object handleUnAutorizedException(UnAuthorizedException e) {
    return new Object(e.getMessage());
  }
}
```



404 에러 같은 경우는 DispatcherServlet에서 발생하는 에러라서 핸들링 할 수 없다.

> Filter단에서 발생하는 예외에 대해서는 핸들링되지 않는다. 예를 들면 404처럼 DispatcherServlet에서 발생하는 예외의 경우 여기서 핸들링되지 않는다.
>
> [404 error redirect in Spring with java confighttps://stackoverflow.com/questions/ask)](https://stackoverflow.com/questions/23574869/404-error-redirect-in-spring-with-java-config)
>
> ```java
> public class WebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
>     @Override
>     protected Class<?>[] getRootConfigClasses() {
>         return null;
>     }
> 
>     /* ... */
> 
>     @Override
>     protected DispatcherServlet createDispatcherServlet(WebApplicationContext servletAppContext) {
>         final DispatcherServlet dispatcherServlet = super.createDispatcherServlet(servletAppContext);
>         dispatcherServlet.setThrowExceptionIfNoHandlerFound(true);
>         return dispatcherServlet;
>     }
> }
> ```
>
> ```java
> @ControllerAdvice
> public class NoHandlerFoundControllerAdvice {
> 
>     @ExceptionHandler(NoHandlerFoundException.class)
>     public ResponseEntity<String> handleNoHandlerFoundException(NoHandlerFoundException ex) {
>         // prepare responseEntity
>         return responseEntity;
>     }
> 
> }
> ```
>
> 위와 같은 방법으로 가능하다고는 하는데 테스트 해보진 않았다.