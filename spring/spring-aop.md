# Spring AOP

관점지향 프로그래밍(Aspect Oriented Programming)

OOP의 공통 기능(로깅, 트랜잭션, 접근 제어 등의 보안) 등에 대한 횡단 영역의 공통된 부분의 중복을 제거하고 모듈화 하는 프로그래밍의 방식

```java
public List<BestCategory> findAll() {
    //log.info("메소드 내용 실행 전 필요한 로그");
    //long start = System.currentTimeMillis();
    List<BestCategory> bestCategories = bestCategoryRepository.findAll();
    //log.info("메소드 내용 실행 완료 후 필요한 로그");
    //long end = System.currentTimeMillis();
    //log.info("메소드 실행 시간 {}", end -start);
    return bestCategories;
}

public Category save(CategoryDto categoryDto) {
    //log.info("메소드 내용 실행 전 필요한 로그");
    //long start = System.currentTimeMillis();
    Category category categoryRepository.save(categoryDto.toEntity());
    //log.info("메소드 내용 실행 완료 후 필요한 로그");
    //long end = System.currentTimeMillis();
    //log.info("메소드 실행 시간 {}", end -start);
    return category;
}
```

위의 코드에서는 메소드의 내용 실행 전 중복으로 들어가는 실행시간 측정, 로깅에 대해서 중복이 발생한다. OOP의 관점에서 상속이나 위임을 사용해 위와같은 기능들의 중복을 제거할수는 있겠지만 깔끔하게 기능들을 모듈화 하기는 어렵다.

OOP의 관점이 아닌 다른 관점으로 공통 요소들을 추출해 모듈화 하기 위해 AOP를 사용한다.

## AOP의 장점

AOP를 사용하면 위와같이 전역적으로 발생할 수 있는 공통 기능들에 대한 관리를 한곳에서 해줄 수 있다.

## AOP의 용어들

AOP 용어들 스프링에서만 사용되는게 아니라 AOP 프레임워크 전체에서 사용되는 용어들.

> ### Target Object
>
> 부가기능을 부여할 대상, ex) CategoryService 등.. 대개 스프링에서는 Service 쪽이 해당.
>
> 스프링에서는 Runtime Weaving을 통해 프록시 된 객체들이 대상 객체에 해당한다.
>
> ### Advice
>
> 실질적인 부가기능을 담은 구현체 Aspect가 언제 적용 될 지를 정의하고 있다.
>
> ```java
> @Around("execution(* codesquad.service.BestCategoryService.findAll(..))")
> ```
>
> 위에서 **@Around에 해당하는 부분이 Advice다**. 필요에 따라 아래와 같은 여러 어노테이션을 통해 적용 시점을 변경할 수 있다.
>
> - @Before
>   - 메소드 실행 전 기능 수행.
> - @After
>   - 메소드 결과와 상관없이 메소드가 완료 된 이후에 기능 수행.
> - @AfterReturning
>   - 메소드가 성공적으로 완료 된 이후에 기능 수행.
> - @AfterThrowing
>   - 메소드 수행 중 예외 발생 시 이후에 기능 수행.
> - @Around
>   - 메소드가 실행되기 전과 후 기능 구행. proceed() 메소드 호출 전, 후를 통해 구분할 수 있다.
>
> ### Pointcut
>
> 부가기능이 적용될 대상을 선정하는 방법을 정의한 모듈, 스프링은 기본적으로 AspectJ 포인트컷 표현식 언어를 사용한다.
>
> Pointcut 표현식에 맞고, JoinPoint에 해당하는 지점에서 해당 Aspect가 실행된다.
>
> ![](/images/spring/pointcut.png)
>
> 포인트컷 표현식은 위와 같은 구조를 가지고 있다. execution은 포인트컷 지정자로 다른 것도 많지만 생략.
>
> ```java
> @Pointcut("execution(* codesquad.service.BestCategoryService.findAll(..))")
> public void findAllBestCategory(){}
> 
> @Pointcut("execution(* codesquad.service.BestCategoryService.find(..))")
> public void findBestCategory(Long categoryId){}
> 
> @Around("findAllBestCategory() || findBestCategory()")
> public Object calculatePerformanceTime(Pr
> ```
>
> 위처럼 @Pointcut을 이용해 재사용 가능한 포인트컷을 등록, 사용할 수도 있고 관계연산자를 이용해 여러 포인트컷을 적용할수도 있다.
>
> ###  Aspect
>
> 부가기능 모듈, 객체지향의 객체처럼 AOP의 한 기능을 가지는 모듈 스프링에서는 @Aspect 어노테이션을 통해 구현할 수 있다.
>
> ### JoinPoint
>
> 메소드 실행이나 예외 처리, 필드값 수정 등에 대한 지점을 나타낸다. Spring AOP에서는 메소드 실행에 대한 JoinPoint만 지원한다.
>
> ### Weaving
>
> Aspect가 지정된 객체를 새로운 프록시 객체를 생성하는 과정.
>
> ![](/images/spring/cglib.png)
>
> 위처럼 CategoryService라는 객체 이외에 CGLIB 프록시 객체가 생성되기 위한 과정을 의미한다.
>
> Compile-time Weaving, Load-time weaving, Run-time weaving 세가지 방식의 Weaving이 존재하고. 스프링 AOP에서는 CGLIB Proxy, JDK Dynamic Proxy를 이용한 Run-time weaving 방식을 제공한다.
>
> ### Proxy
>
> 타겟을 감싸서 요청을 대신 받아주는 랩핑 클래스. Weaving을 통해서 Proxy 객체를 생성하며 Spring AOP Proxy는 CGLIB Proxy, JDK Dynamic Proxy를 사용한다. 스프링의 AOP는 이 프록시 객체를 통해 작동하게 된다.

## Spring Proxy

타겟을 감싸서 요청을 대신 받아주는 랩핑 클래스. Weaving을 통해서 Proxy 객체를 생성하며 Spring AOP Proxy는 CGLIB Proxy, JDK Dynamic Proxy를 사용한다. 스프링의 AOP는 이 프록시 객체를 통해 작동하게 된다. 

간략하게 돌아가는 구조를 예로 들면..

```java
@Service
public class CategoryService {
    @Autowired
    private CategoryRepository categoryRepository;

    @Transactional
    public Category save(CategoryDto categoryDto) {
        return categoryRepository.save(categoryDto.toEntity());
    }
}
```

위의 Service의 save를 호출했을 때, CategoryService로 바로 접근하는것이 아니라.

![](/images/spring/cglib.png)

$$EnhancerBySpringCGLIB 라고 적힌 weaving으로 생성된 CGLIB 프록시를 통해 간접적으로 접근하게 된다. 저 프록시 객체를 통해서 트랜잭션이나 로깅 등 AOP와 관련된 처리가 동작하게 된다.

내부적으로 어떻게 동작하는지는 정확히는 모르겠지만.. 트랜잭션의 예를 들면 CGLIB 프록시 객체를 이용해서 아래처럼 동작하지 않을까? 안 까봐서 모르겠다.

```java
public class CategoryServiceEnhancerBySpringCGLIB {
    
    private CategoryService categoryService;
    
    public CategoryServiceEnhancerBySpringCGLIB(CategoryService categoryService) {
        this.categoryService = categoryService;
    }

    public void save(CategoryDto categoryDto) {
        TransactionManager transactionManager = new TransactionManager();
        try {
            this.categoryService.save(categoryDto);
            //CategoryService를 호출해 로직 실행 후, 에러가 없으면 commit 실행
            transactionManager.commit();
        } catch (Exception e) {
            transactionManager.rollBack();
            throw e;
        }
    }
}
```
Spring AOP의 Proxy는 CGLIB Proxy, JDK Dynamic Proxy 두가지 종류가 있다.
![](/images/spring/springaop-process.png)

과거에는 기본적으로 **인터페이스가 있고, 그의 구현체가 있는 클래스의 경우 JDK dynamic Proxy를 사용**하고 **인터페이스가 없는 경우 CGLIB Proxy를 사용**했다.

> CGLIB Proxy는 상속을 통해 Proxy를 구현하기 때문에 final 클래스인 경우 Proxy를 생성할 수 없다. kotlin의 경우에 모든 클래스들이 default로 final이기 때문에 AOP가 제대로 잘 작동 안한다고 들었는데 잘 모르겠다.

Spring Boot에서는 이제 **디폴트로 CGLib Proxy를 생성**하는 것 같다. 성능도 더 좋고 예외도 적다고 하니 굳이 CGLib Proxy를 사용하지 않을 이유가 없을 듯?

 JDk Dynamic Proxy는 Java Reflection을 이용해 조금 속도가 느리다고 한다. JDK Dynamic Proxy를 강제로 사용하려면 아래와 같은 설정을 추가하면 된다.

```java
 @EnableAspectjAutoProxt(proxyTargetClass = false)
 @Configuration
 public class ProxyConfig {
     
 }
```

그리고 예외적으로 **spring-data-jpa에서는 JDK Dynamic Proxy를 사용**해 Repository를 생성한다. 이유는 나도 잘 모르겠다.

## AOP Weaving의 종류

Weaving이란 지정된 객체에 Aspect를 부여해서 새로운 프록시 객체를 생성하는 과정을 의미한다. 위에서 자동으로 만들어진 **$$EnhancerBySpringCGLIB 같은 래핑된 클래스를 생성하는 과정**을 말한다.

아래와 같은 3가지 방법의 Weaving을 제공한다. Spring AOP는 기본적으로 무조건 Run-time weaving으로 동작한다.

> ### Run-time weaving
>
> Spring AOP에서 사용하는 weaving 방식. **스프링에서는 Run-time weaving을 통해 CGLIB Proxy 혹은 JDK Dynamic Proxy를 생성**한다.
>
> ### Load-time weaving
>
> 일반적으로 사용되는 Spring AOP가 아닌  AspectJ 라이브러리를 추가하여 사용해야 한다.
>
> 객체를 Load 할때, AspectJ에 의해서 weaving된 객체를 넘겨주는 방식.
>
> applicationContext에 로드된 객체들을 불러온 뒤, Aspectj weaver에 의해 객체들을 weaving한다고 함. 객체들을 다 불러온 뒤 weaving을 하기 때문에 약간의 퍼포먼스 하락이 있다고 한다.
>
> ### Compile-time weaving
>
> 일반적으로 사용되는 Spring AOP가 아닌  AspectJ 라이브러리를 추가하여 사용해야 한다.
>
> Compile 시에 Aspectj에서 필요한 객체 weaving을 통해 클래스를 생성하는 방식이다. 위의 Load-time에 대한 절차가 없어서 퍼포먼스 하락 없이 구성이 가능하다.
>
> 다만 Lombok과 같이 compile시 간섭하는 plugin들과 충돌이 발생한다고 함. 자바 8 기반 코드들에서 Lombok을 사용하지 않는 경우는 거의 없다고 봐도 되기 때문에 사실상 사용이 힘들듯?

## Spring AOP와 AspectJ의 차이점

Spring에서 지원하는 AOP와 AspectJ를 사용하는것과 차이점이 조금 있는데. Spring AOP는 메소드 실행에 대한 JoinPoint만 제공하고, runtime weaving만 지원, 그리고 실행속도에 있어서도 AspectJ를 사용하는것보다 느린 여러 단점을 가지고 있다.

AspectJ가 실행속도 측면이나 기능적인 측면에서 더 좋다고 할 수 있지만 AspectJ에 대해서 대충만 훑어봐도 너무 어려워서 Spring AOP를 이용해 개발을 하고 그 이상의 기능이 필요한 경우 AspectJ를 확장해서 사용하는 것이 좋을 것 같다.

![](/images/spring/spring-aop-aspectj-diff.png)
[baeldung spring-aop vs aspectj](https://www.baeldung.com/spring-aop-vs-aspectj)

## Spring AOP 를 이용한 실행시간 측정

간단한 코드를 통해 실행시간 측정을 AOP를 이용해 테스트해보자.

BestCategoryLogingAspect.java

```java
@Component
@Aspect
public class BestCategoryLogingAspect {

    @Around("execution(* codesquad.service.BestCategoryService.findAll(..))")
    public Object calculatePerformanceTime(ProceedingJoinPoint proceedingJoinPoint) {
        Object result = null;
        try {
            long start = System.currentTimeMillis();
            result = proceedingJoinPoint.proceed();
            long end = System.currentTimeMillis();

            System.out.println("수행 시간 : "+ (end - start));
        } catch (Throwable throwable) {
            System.out.println("exception! ");
        }
        return result;
    }
}
```

BestCategoryService.java

```java
@Service
public class BestCategoryService {
    public static final Logger log = LoggerFactory.getLogger(MemberController.class);

    @Autowired
    private BestCategoryRepository bestCategoryRepository;

    public List<BestCategory> findAll() {
        return bestCategoryRepository.findAll();
    }
}
```

위와 같이 설정하고, 코드를 실행시켜 보면 따로 코드를 설정해 준 게 없는데도 AOP가 작동하면서 수행 시간이 찍히게 된다.

![](/images/spring/exetime.png)

 [Spring AOP 이동욱님 블로그](https://jojoldu.tistory.com/71)

## 내부 메소드 호출과 Proxy

아래와 같은 코드가 있다고 했을 때, methodB에 적용된 @Around는 작동하지 않는다고 한다. 테스트는 안해봤는데 Proxy의 동작 원리를 생각하면 작동하지 않는게 당연한것 같다.

이외에도 Propagation.REQUIRES_NEW 설정을 통해 아래처럼 MethodA에서 MethodB를 호출해도 새 트랜잭션으로 묶이지 않고 동일한 트랜잭션으로 묶이는 문제가 있다고 한다.

```java
public class ExampleService {
    public void methodA(){
        //.........
        this.methodB();
        //...........
    }
    public void methodB(){
        //...............
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
	public void transactionMethodA(){
        //...............
		transactionMethodB();
	}
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void transactionMethodB(){
        //...............
	}
}

@Component
@Aspect
public class BestCategoryLogingAspect {
    @Pointcut("execution(* methodA(..))")
    public void methodA(){}
    
    @Pointcut("execution(* methodB(..))")
    public void methodB(){}

    @Around("execution(* methodA(..)) || execution(* methodB(..))")
    public Object calculatePerformanceTime() {
        //...............
    }
}
```

위에서 this.methodB()로 호출하는 경우 Proxy 객체를 이용해 호출하는 것이 아니기 때문에 AOP가 제대로 작동하지 않는다.

```java
public class ExampleService {
    @Autowired
    private ExampleService exampleService;
    public void methodA(){
        //.........
        exampleService.methodB();
        //...........
    }
    public void methodB(){
        //...............
    }
    @Transactional(propagation = Propagation.REQUIRES_NEW)
	public void transactionMethodA(){
        //...............
		exampleService.transactionMethodB();
	}
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void transactionMethodB(){
        //...............
	}
}
```

위와 같은 방법으로 하면 해결된다고 한다. 아니면 Run-time weaving이 아닌 Compile-time weaving이나 Load-time weaving을 사용하는 AspectJ 라이브러리를 활용해도 해결할 수 있다고 함. 실제로 해보진 않았지만 구동 방식상 정상적으로 작동하지 않을까 싶음.

[stackoverflow spring aop not working for method call inside another method](https://stackoverflow.com/questions/13564627/spring-aop-not-working-for-method-call-inside-another-method)

아래 글에 AspectJ를 활용해 개선하는 방법에 대해서 나와있다.

[Wan Blog Spring Transactional AspectJ Compile](https://minwan1.github.io/2017/10/29/2017-10-29-Spring-Transaction,AspectJ-Compile/)



### 참고

- [AOP 정리 이동욱님 블로그](https://jojoldu.tistory.com/71)
- [Spring 프레임워크 AOP](http://www.javajigi.net/pages/viewpage.action?pageId=1068)
- [[Spring 레퍼런스] 8장 스프링의 관점 지향 프로그래밍 #1](https://blog.outsider.ne.kr/843)
- [Bealdung Aspectj](https://www.baeldung.com/aspectj)
- [Wan Blog Spring AOP Proxy](https://minwan1.github.io/2017/10/29/2017-10-29-Spring-AOP-Proxy/)
- [Wan Blog Spring Transactional AspectJ Compile](https://minwan1.github.io/2017/10/29/2017-10-29-Spring-Transaction,AspectJ-Compile/)
- [stackoverflow spring aop not working for method call inside another method](https://stackoverflow.com/questions/13564627/spring-aop-not-working-for-method-call-inside-another-method)
