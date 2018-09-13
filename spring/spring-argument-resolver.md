## Spring 커스텀 HandlerMethodArgumentResolver 적용하기

HandlerMethodArgumentResolver 인터페이스를 구현해서 커스텀 ArgumentResolver를 만들어보자.

Spring Controller단에서 파라미터를 수정하거나 변경하는 경우에 사용한다.

예를 들어, 로그인 사용자의 아이디를 가져온다고 생각해보자. 매번 HttpSession을 파라미터로 받아서 세션에서 유저 정보를 꺼내준다거나 하는 식으로 사용해야 한다.

그런 경우가 한 두번이라면 모를까, 매번 체크해주어야 하는 것이기 때문에 번거롭다. HandlerMethodArgumentResolver는 사용자 요청이 Controller에 도달하기 전에 그 요청의 파라미터들을 수정할 수 있게 해주는 역할을 한다.



### HttpSession의 유저 정보를 @LoginUser 어노테이션을 이용해 받아오기

`@LoginUser`라는 어노테이션을 구현해 현재 세션의 로그인 유저 정보를 받아오는 것을 작성한다고 생각해보자.

우선 LoginUser 어노테이션을 생성한다.

> LoginUser annotation

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface LoginUser {
}
```

> LoginUserHandlerMethodArgumentResolver.java

HandlerMethodArgumentResolver 인터페이스를 구현하는 커스텀 ArgumentResolver 클래스를 생성한다.

이때 두가지 메소드를 구현해주어야 한다.

```java
public class LoginUserHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {
	@Override
	public Object resolveArgument(MethodParameter arg0, ModelAndViewContainer arg1, NativeWebRequest webRequest,
			WebDataBinderFactory arg3) throws Exception {
		Object user = webRequest.getAttribute(UserSessionUtils.USER_SESSION_KEY, WebRequest.SCOPE_SESSION);
		return user;
	}
	@Override
	public boolean supportsParameter(MethodParameter parameter) {
		return parameter.hasParameterAnnotation(LoginUser.class);
	}
}
```

- **supportsparameter method**

Resolver가 적용 가능한지 검사하는 역할을 한다. 여기서는 해당 컨트롤러의 파라미터에 LoginUser 어노테이션이 붙어있는지를 검사해서 boolean 값을 리턴한다. `@LoginUser` 가 붙어있다면 reolsveArgument라는 메소드가 작동하는 방식인 것 같다.

> 같은 타입인지를 체크하려면 `CommandMap.class.isAssignableFrom(parameter.getParameterType());` 이와 같이 처리할 수도 있는것 같다.

- **resolveArgument method**

메서드는 파라미터와 기타 정보를 받아서 실제 객체를 반환한다. 여기서는 webRequest에 담긴 세션에 담긴 유저 정보를 가져온다. 

`WebRequest.SCOPE_SESSION` 을 통해서 Request의 세션을 가져오는 것 같다.

> #### SCOPE_SESSION
>
> ```
> static final int SCOPE_SESSION
> ```
>
> Constant that indicates session scope.
>
> This preferably refers to a locally isolated session, if such a distinction is available (for example, in a Portlet environment). Else, it simply refers to the common session.

## Custom ArgumentResolver 등록하기

위에서 만든 Custom ArgumentResolver는 만들기만 해서 작동하지 않고, 등록하는 과정을 따로 거쳐야 한다.

```java
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {
    @Bean
    public LoginUserHandlerMethodArgumentResolver loginUserMethodArgumentResolver() {
    		return new LoginUserHandlerMethodArgumentResolver();
    }
    
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
    		argumentResolvers.add(loginUserMethodArgumentResolver());
    }
}
```

위와 같이 등록하는 과정을 거친 후 아래처럼 사용할 수 있다.

```java
@GetMapping("/{id}")
public NoteDto show(@LoginUser User loginUser) {
	//
}
```

위와같이 어노테이션만 붙여주면, 현재 로그인 된 유저 정보가 자동으로 들어가게 된다.