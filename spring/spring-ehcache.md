## Spring Ehcache 적용하기

잘 변경되지 않으면서 반복적으로 호출되는 데이터에 대해 캐싱을 적용해서 서버의 성능을 향상시킬 수 있다.

기본적으로 브라우저(크롬, 익스 등)단에서 캐싱을 적용해주고 있지만 그 이전 단계인 서버에서 캐싱을 적용한다면 DB 조회를 줄이면서 서버의 부하를 조금 더 줄일 수 있다.

Spring에서는 **spring-starter-cache**를 통해 로컬 캐싱을 사용할 수 있는데 기본적으로 ConcurrentHashMap을 사용해 제공하고 있다. 기본 캐시매니저를 사용하지 않고 EhCache를 적용하려면 두가지 의존성을 추가해야 한다.

```xml
compile('org.springframework.boot:spring-boot-starter-cache')
compile group: 'net.sf.ehcache', name: 'ehcache', version: '2.9.0'
```

`spring-boot-starter-cache` 를 추가하면 캐시 관련 의존성(CacheManager 등)을 직접 추가하지 않아도 된다.

spring-starter-cache에서는 기본적으로 `ConcurrentHashMap` 을 사용하는데 이를 `Ehcache` 를 사용하도록 변경하기 위해서 아래의 ehcache 의존성을 추가했다.

기본적으로 ehcache 의존성이 적용되기 전 CacheManager는`ConcurrentMapCacheManager` 로 설정되고 ehcache 의존성을 적용하고 나면 `EhCacheCacheManager` 로 변경, 자동으로 적용된다.



캐싱 설정 정보는 디폴트로 resources 아래의 `ehcache.xml` 파일을 보는데 아래처럼 직접 지정해 줄 수 있다. 변경하고 싶으면 해당 파일 네임을 지정해주면 된다.

```xml
# application.yml
spring:
  cache:
    ehcache:
      config: classpath:ehcache.xml
      
      
# application.properties
spring.cache.ehcache.config=classpath:ehcache.xml

```



CacheManager가 디폴트 캐시매니저인지 Spring EhcacheManager인지 실행될 때 체크하기 위해서 CommandLineRunner 추가, 캐싱을 사용하기 위해 @EnableCaching 어노테이션을 추가하자.

**따로 빈을 만들어 주지 않아도 CacheManager, EhCacheManagerFactoryBean이 생성되게 된다.(하지만 이렇게 디폴트 설정으로 사용 하면 테스트 코드가 동작할 때 CacheManager가 여러번 생성되는지 에러가 발생한다.)**

테스트 코드가 돌면서 여러개의 CacheManager를 생성하기 때문인 것 같다.

```java
@Configuration
@EnableCaching
public class CacheConfig implements CommandLineRunner {
    private static final Logger log = LoggerFactory.getLogger(CacheConfig.class);
    @Autowired
    private CacheManager cacheManager;

    @Override
    public void run(String... args) throws Exception {
        log.info("\n\n" + "=========================================================\n"
                + "Using cache manager: " + this.cacheManager.getClass().getName() + "\n"
                + "=========================================================\n\n");
    }
}
```

위와 같은 디폴트 설정 이외에 직접 CacheManager를 빈으로 등록할 수 있다.

디폴트 설정을 사용하면 테스트 코드 실행 시 CacheManager exception이 발생하니까 아래 방법으로 빈을 직접 설정하고 setShared 설정을 true로 두고 사용하자. 

shared 설정은 cacheManager 싱글톤 여부를 지정하는 값이라고 하는데. cacheManager 인스턴스가 없으면 생성하도록 true 값을 지정하자. 이러면 테스트 코드를 돌릴 때, 이미 인스턴스가 존재하는경우 생성하지 않고 기존의 인스턴스를 사용해서 그런지 에러가 발생하지 않는다. 

```java
@Configuration
@EnableCaching
public class CacheConfig implements CommandLineRunner {
    private static final Logger log = LoggerFactory.getLogger(CacheConfig.class);
    @Autowired
    private CacheManager cacheManager;

    @Bean
    public CacheManager cacheManager() {
        return new EhCacheCacheManager(ehCacheCacheManager().getObject());
    }

    @Bean
    public EhCacheManagerFactoryBean ehCacheCacheManager() {
        EhCacheManagerFactoryBean cmfb = new EhCacheManagerFactoryBean();
        cmfb.setConfigLocation(new ClassPathResource("ehcache.xml"));
        cmfb.setShared(true);
        return cmfb;
    }

    @Override
    public void run(String... args) throws Exception {
        log.info("\n\n" + "=========================================================\n"
                + "Using cache manager: " + this.cacheManager.getClass().getName() + "\n"
                + "=========================================================\n\n");
    }
}
```



## ehcache.xml 파일 작성

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
         updateCheck="false">
    <diskStore path="java.io.tmpdir"/>
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            overflowToDisk="true"
            maxElementsOnDisk="10000000"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU"/>
    <cache name="findByBestCategoryCache"
           maxEntriesLocalHeap="10000"
           maxEntriesLocalDisk="1000"
           eternal="false"
           diskSpoolBufferSizeMB="20"
           timeToIdleSeconds="300" timeToLiveSeconds="600"
           memoryStoreEvictionPolicy="LFU"
           transactionalMode="off">
        <persistence strategy="localTempSwap"/>
    </cache>
</ehcache>
```

설정을 조금 살펴보면, `<defaultCache>` 는 코드 내에서 정의한 캐시에 대한 디폴트 값을 의미한다. `<cache>` 는 하나의 캐시에 대한 설정을 지정할 때 사용한다고 한다. 자세한 내용은 생략.

- **maxEntriesLocalHeap**

캐시가 로컬 힙 메모리에서 사용할 수있는 캐시 항목 또는 바이트의 최대값

- **timeToIdleSeconds**

Element가 지정한 시간 동안 사용(조회)되지 않으면 캐시에서 제거된다. 이 값이 0인 경우 조회 관련 만료 시간을 지정하지 않는다. 기본값은 0이다. 

- **timeToLiveSeconds**

Element가 존재하는 시간. 이 시간이 지나면 캐시에서 제거된다. 이 시간이 0이면 만료 시간을 지정하지 않는다. 기본값은 0이다.

- **memoryStoreEvictionPolicy**

객체의 개수가 maxElementsInMemory에 도달했을 때,모메리에서 객체를 어떻게 제거할 지에 대한 정책을 지정한다. 기본값은 LRU이다. FIFO와 LFU도 지정할 수 있다. 

나머지는 알아서 잘 알아보자.

## @Cacaheable, @CacheEvict

스프링 starter-cache에서 지원해주는 어노테이션 같은데 위 어노테이션을 사용해서 간편하게 캐싱과 캐시 삭제를 할 수 있다.

- **@Cacheable**

```java
@Cacheable(value="usercache", key = "#root.methodName")
public List<Data> findAll() {
    return dataRepository.findAll();
}
```

위와 같은 코드에 `@Cacheable` 을 지정하고 밸류와 키를 지정해주면, 저 밸류와 키를 가진 캐시를 생성하고 캐시가 존재한다면 저 메소드가 다시 호출 될 떄 안의 메소드를 호출하지 않고 캐싱 된 값을 가져온다.

key를 따로 지정해주지 않으면 메소드 파라미터가 키 값이 되며, 메소드 파라미터가 없다면 0이 키 값이 된다. `@CacheEvict` 할 때 참고 할 것.  [참고한 관련 링크](http://javaiyagi.tistory.com/352)

파라미터가 없을 시 임의로 스트링 키 값을 지정해 줄 순 없다. 아래와 같은 방법으로 메소드 네임이나 static한 값을 지정해 줄 순 있음. [관련 StackOverFlow 링크](https://stackoverflow.com/questions/33383366/cacheble-annotation-on-no-parameter-method/33384311)

```java
@Cacheable(value="usercache", key = "#root.methodName")

public static final String MY_KEY = "mykey";
@Cacheable(value="usercache", key = "#root.target.MY_KEY")
```

- **@CacheEvict**

위에랑 똑같다. 해당 메소드 수행 시 캐시를 삭제하는 어노테이션 키 밸류 값을 가진 캐시를 삭제한다.

```java
@CacheEvict(value = "findMemberCache", key="#id")
public Data update(Long id) {
    return dataRepository.findById(id);
}
```

## CacheManager already exists in the same VM

캐싱 관련 빈 설정을 디폴트로 두고 사용하다가 테스트 코드 동작 과정에서 아래와 같은 에러 발생시

```
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'ehCacheCacheManager' defined in class path resource [org/springframework/boot/autoconfigure/cache/EhCacheCacheConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [net.sf.ehcache.CacheManager]: Factory method 'ehCacheCacheManager' threw exception; nested exception is net.sf.ehcache.CacheException: Another unnamed CacheManager already exists in the same VM. Please provide unique names for each CacheManager in the config or do one of following:
1. Use one of the CacheManager.create() static factory methods to reuse same CacheManager with same name or create one if necessary
2. Shutdown the earlier cacheManager before creating new one with same name.

Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [net.sf.ehcache.CacheManager]: Factory method 'ehCacheCacheManager' threw exception; nested exception is net.sf.ehcache.CacheException: Another unnamed CacheManager already exists in the same VM. Please provide unique names for each CacheManager in the config or do one of following:
```

> Versions of Ehcache before version 2.5 allowed any number of CacheManagers with the same name (same configuration resource) to exist in a JVM.
>
> Ehcache 2.5 and higher does not allow multiple CacheManagers with the same name to exist in the same JVM. CacheManager() constructors creating non-Singleton CacheManagers can violate this rule

맨 위에 적어둔 것 처럼 CacheManager의 생성 공유 `EhCacheManagerFactoryBean`  shared 옵션을 true로 두면 된다.

```
@Bean
public EhCacheManagerFactoryBean ehCacheCacheManager() {
	EhCacheManagerFactoryBean cmfb = new EhCacheManagerFactoryBean();
	cmfb.setConfigLocation(new ClassPathResource("ehcache.xml"));
	cmfb.setShared(true);
	return cmfb;
}
```
[관련 StackOverflow 문서 링크](https://stackoverflow.com/questions/10013288/another-unnamed-cachemanager-already-exists-in-the-same-vm-ehcache-2-5)



shared 설정은 cacheManager 싱글톤 여부를 지정하는 값이라고 하는데. cacheManager 인스턴스가 없으면 생성하도록 true 값을 지정하자. Shared 속성을 바꾸지 말고 acceptExisting 설정을 바꿔도 정상적으로 동작할 거라고 하는데 테스트는 안해봤다. acceptExisting는 이름 중복 사용여부인데, 동일한 이름이 있는 경우 생성하지 않도록 하는 속성이다. [관련 내용 링크](http://syaku.tistory.com/322)



참고

- [창천항로님의 블로그](http://jojoldu.tistory.com/57)
- [최범균님 블로그](http://javacan.tistory.com/entry/133)
