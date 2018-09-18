# Jpa Auditing을 통해 엔티티 중복 제거, 자동화하기

JPA 엔티티를 생성하다보면 중복되는 부분들이 많이 발생한다. 등록 시간, 수정 시간, id 등등... 그 외에도 등록 시간과 수정 시간은 엔티티 수정, 생성 시 마다 직접 입력해주어야 한다.

중복되는 부분은 Auditing Entity를 통해 중복을 제거하고 AuditingEntityListener를 통해 생성 시간, 수정 시간, 생성자 정보 등을 자동으로 넣어줄 수 있다.

아래와 같이 id, 생성 시간, 수정 시간, 생성자, 수정자에 대한 정보를 가지는 Superclass의 역할을 할 엔티티를 생성하자.

- **AuditorEntity.java**

```java
import lombok.Getter;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.Column;
import javax.persistence.EntityListeners;
import javax.persistence.ManyToOne;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@MappedSuperclass
@EntityListeners(value = { AuditingEntityListener.class })
@Getter
public abstract class AuditorEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    protected Long id;
    @Column(nullable = false, updatable = false)
    @CreatedDate
    private LocalDateTime createdDate;
    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
    @ManyToOne
    @CreatedBy
    private Account createdBy;
    @ManyToOne @LastModifiedBy
    private Account lastModifiedBy;
}
```

그리고, @EnableJpaAuditing 어노테이션을 추가해 JpaAuditing 설정을 사용하도록 하자.

- **JpaAuditingConfig.java**

```java
@EnableJpaAuditing
@Configuration
public class JpaAuditingConfig {

}
```

위와같이 설정하고 아래와 같이 상속해서 엔티티를 생성하면 그 엔티티에 대해서 createDate와 modifiedDate에 대해서 자동으로 변경해주게 된다. 

> ex) 아래와 같이 Comment 엔티티를 생성할 때, AuditorEntity를 상속해서 만들 수 있다. 그럼 id와 생성 시간, 수정 시간 및 생성자나 수정자에 대한 정보를 따로 추가해주지 않아도 된다.

```java
public class Comment extends AuditorEntity {
}
```

createdBy나 lastModifiedBy도 자동으로 설정할 수 있는데, 이는 아래와 같은 추가적인 설정을 더 넣어야 한다.

우선 설정 파일에 auditorAwareRef를 추가한다.

```java
@EnableJpaAuditing(auditorAwareRef = "securityAuditorAware")
@Configuration
public class JpaAuditingConfig {

}
```

그리고 누구에 의해서 작성, 수정되었는지를 가져올 securityAuditorAware를 생성하자.

```java
@Component
public class SecurityAuditorAware implements AuditorAware<Account> {
    @Autowired
    private AccountService accountService;
    @Override
    public Optional<Account> getCurrentAuditor() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if(authentication instanceof AnonymousAuthenticationToken) {
            return Optional.empty();
        }
        PostAuthorizationToken postAuthorizationToken = (PostAuthorizationToken)authentication;
        return Optional.ofNullable(accountService.findByEmail(postAuthorizationToken.getAccountContext().getAccount().getEmail()));
    }
}
```

일반적으로 세션을 사용하는 로그인의 경우에는 적용되지 않는 방법이고 스프링 시큐리티를 사용한 경우 위와 같은 방법으로 가져올 수 있다. 세션 사용 기반에서는 세션이 아니라 스레드 로컬에 저장하도록 하는식으로 해서 구현할 수 있을것 같은데.. 안해봐서 잘 모르겠다.

그리고 추가적으로 LocalDateTime을 사용하기 때문에 Serialize, DeSerialize 할때 문제가 발생할 가능성이 높다. 이전 포스팅에서 만들었던 커스텀 Serializer나 DeSerializer를 만들고 LocalDateTime을 사용하는 필드위에 지정해주자.

>  ex)

```java
@JsonSerialize(using = CustomLocalDateTimeSerializer.class)
@JsonDeserialize(using = CustomLocalDateTimeDeserializer.class)
@CreatedDate
private LocalDateTime createdDate;
```

