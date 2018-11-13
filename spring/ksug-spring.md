# 스프링 기반 REST API 개발

- [백기선님 slideshare](https://www.slideshare.net/whiteship/rest-api-development-with-spring?fbclid=IwAR2w-1u0Gih9IOrBSo9Q8dunbT4W6olQ-RGbwHdbY43j2t1I25aMaU7JySE)

- [백기선님 study repository](https://github.com/keesun/study/tree/master/ksug201811restapi)

백기선님의 스프링 기반 REST API 개발 세미나 관련 정리.

앞으로 자바 개발을 할 일이 있을진 모르겠지만 REST API와 관련된 HATEOAS나 Self-Descriptive와 같은 개념은 어떤 언어를 사용해 개발하든 적용될 것 같아서 한번 처음부터 진행해봤다.

## REST API

일반적으로 우리가 REST API라고 하면서 제공하는건 사실 제대로 된 REST API의 형태를 만족하지 않는다.

여러 요건들을 충족해야 하지만 그중에서도 거의 90% 이상이 아래 두가지 요건은 충족하지 못하고 있다.

### Self-Descriptive

```json
HTTP/1.1 200 OK
ContentType: application/json

{
    "data": "hello"
}
```

일반적으로 위와 같이 API를 만들어 내려주는 경우가 많을텐데 위 정보는 json 안의 값들이 어떤 의미를 가지는지 알수 없어서 Self-Descriptive하지 않다.

```html
GET /todos HTTP/1.1
Content-Type: text/html

<html>
    <body>
        <a href="http://todos/1">집에 가기</a>
    </body>
</html>
```

GET 요청으로 받아온 html같은 경우에는 ContentType을 보고 text/html임을 확인할 수 있고

정의된 HTTP 명세를 통해 해당 내용이 어떤 값들을 제공하는지를 알 수 있다.

#### Self-Descriptive를 만족하는 REST API 제공 방법

내용만을 보고 어떤 데이터를 내려주는지에 대해 Self-Descriptive를 만족시키기 위한 두가지 방법이 존재한다.

- **Media Type 정의, IANA에 등록하기**

데이터에 대한 Media Type을 직접 IANA에 등록해야 하는데 새로운 데이터를 내려줄 때 마다 등록해주어야 함. 번거로워서 아무도 하지 않을 듯.

- **Profile Link Header 추가하기**

json 명세에 대한 문서를 바디나 헤더에 첨부한다. 아직 브라우저에서 지원하지 않는 경우가 많아 아래처럼 헤더가 아닌 바디에 직접 링크를 넣어줄 수 있다.

```json
{
    "_links": {
        "self": { "href": "https://api.example.com/player/1234567890" },
        "friends": { "href": "https://api.example.com/player/1234567890/friends" }
    }
}
```

#### HATEOAS(Hypermisa as the engine of application state)

HATEOAS를 만족하기 위해서는 이벤트 목록을 받아오는 API 명세에 이벤트를 생성하거나, 업데이트 할 수 있는 링크들이 포함되어야 한다.

```html
GET /todos HTTP/1.1
Content-Type: text/html

<html>
    <body>
        <a href="http://todos/1">너무 졸려</a>
    </body>
</html>
```

위의 GET 요청으로 받아온 html 형태의 경우 a 태그를 통해 해당 todos에 관련된 작업을 수행할 수 있어 위의 내용 하나만으로 완전하게 HATEOAS를 만족한다고 할 수 있다.

```json
HTTP/1.1 200 OK
ContentType: application/json

{
    "todos": [
        {
        "id": "1",
        "text": "집에 가기"
        },
        {
        "id": "2",
        "text": "집에 안 가기"
        }
    ]
}
```

위의 json의 경우 명세를 통해 구조 분석이 가능하고 값이 들어있지만 todos와 관련된 작업을 수행할 수 있는 url이 응답에 포함되어 있지 않기 때문에 HATEOAS를 만족하지 않는다.

#### HATEOAS를 만족하는 REST API를 제공하는 방법

- **Data에 다양한 방법으로 하이퍼링크 표현**

JSON API 나 HAL 등 다양한 방법을 통해 body에 링크 정보를 넣어준다.

```json
{
    "_links": {
        "self": { "href": "https://api.example.com/player/1234567890" },
        "friends": { "href": "https://api.example.com/player/1234567890/friends" }
    }
}
```

- **Link Header나 Location을 제공한다.**

자세한 내용은 찾아보지 않았음.

## Event 엔티티를 통해 REST API 구현하기

위의 Self-Descriptive와 HATEOAS에 맞는 REST API를 직접 구현해보자.

Spring Initializer를 이용해 아래와 같은 의존성을 추가하고 프로젝트를 생성하자.

![image-20181110224830905](/images/spring/image-20181110224830905.png)

### Event 도메인 정의하기

[Event 도메인 정의 커밋](https://github.com/tramyu/ksug-rest-api/commit/92591448d432ddd665a7a4ea5b217fbee278495c)

Event

```java
@Entity
@Getter
@Setter
@EqualsAndHashCode(of = "id")
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Event {
    @Id
    @GeneratedValue
    private Integer id;
    private String name;
    private String description;
    private LocalDateTime beginEnrollmentDateTime;
    private LocalDateTime closeEnrollmentDateTime;
    private LocalDateTime beginEventDateTime;
    private LocalDateTime endEventDateTime;
    private String location; // (optional) 이게 없으면 온라인 모임
    private int basePrice; // (optional)
    private int maxPrice; // (optional)
    private int limitOfEnrollment;
    private boolean offline;
    private boolean free;
    @Enumerated(EnumType.STRING)
    private EventStatus eventStatus = EventStatus.DRAFT;
}

```

EventStatus

```java
public enum EventStatus {
    DRAFT, PUBLISHED, ENROLLMENT_STARTED
}
```

EventTest

```java
package com.tram.restapi.domain;

import org.junit.Test;

import static org.assertj.core.api.Assertions.assertThat;


public class EventTest {
    @Test
    public void createEventByBuilderTest() {
        Event event = Event.builder()
                .name("Event")
                .description("스프링 이벤트")
                .build();
        assertThat(event.getName()).isNotEmpty();
    }

    @Test
    public void createEventByDefaultConstructor() {
        Event event = new Event();
        event.setName("Event");
        event.setDescription("스프링 이벤트");
        assertThat(event.getName()).isNotEmpty();
    }
}
```

> - JPA 무한 참조 문제와 EqualsAndHashCode
>
> ```java
> @EqualsAndHashCode(of = "id")
> ```
>
> 모든 필드를 다 사용하는데 id로만 비교하게 함. 순환 참조를 통한 stackOverFlow 방지용으로 id로만 비교하게 한다. id는 유니크라서 아이디로만 비교해도 무방할 것 같다.
>
> - Builder 패턴 사용과 유연한 객체 설계
>
> Builder 패턴을 통해 가독성을 증가시키고 객체 설계를 유연하게 한다. 하지만 default 생성자를 사용할 수 없게 되어 자바 기본 스펙으로 객체 생성이 어려워진다. AllArgs, NoArgs를 추가해주자.
>
> ```java
> @Builder
> @AllArgsContsructor
> @NoArgsConstructor
> public class Event {
> 	// 생략
> }
> ```
>  추가로 롬복 사용에 따른 코드 커버리지 감소 문제가 있는데 이 또한 Lombok과 유사한 방식으로 컴파일 타임에 롬복 Getter, Setter 관련된 테스트 코드를 작성해주는 어노테이션을 직접 개발하거나 하는 방식으로 코드 커버리지를 확보 가능하다. 근데 번거로울 것 같음.. 그리고 추가적으로 @Data 어노테이션을 사용하는건 지양하자.
>
> - Meta Annotation
>
> Custom Annotation에 메타 Annotation을 붙여서 설정을 포함시킬 수 있다고 함.

## Event 생성 API 정의, 테스트 코드 작성하기

[Event 생성 API 정의, 테스트 코드 작성하기 커밋](https://github.com/tramyu/ksug-rest-api/commit/98b756820ed73c03ba8e1977304e76b202e779df)

- **ModelMapper 의존성 추가하기**

Dto -> Event Domain 객체와의 전환을 위한 modelmapper 추가

```groovy
dependencies {
	compile group: 'org.modelmapper', name: 'modelmapper', version: '2.3.1'
}
```

사용법

```java
Event event = modelMapper.map(eventDto, Event.class);
```

- **EventDto**

```java
@Getter
@Setter
@Builder @AllArgsConstructor @NoArgsConstructor
public class EventDto {
    @NotEmpty
    private String name;
    @NotEmpty
    private String description;
    @NotNull
    private LocalDateTime beginEnrollmentDateTime;
    @NotNull
    private LocalDateTime closeEnrollmentDateTime;
    @NotNull
    private LocalDateTime beginEventDateTime;
    @NotNull
    private LocalDateTime endEventDateTime;
    private String location; // (optional) 이게 없으면 온라인 모임
    @Min(0)
    private int basePrice; // (optional)
    @Min(0)
    private int maxPrice; // (optional)
    @Min(0)
    private int limitOfEnrollment;
}

```

- **EventRepository**

```java
public interface EventRepository extends CrudRepository<Event, Long> {
}

```

- **EventController**

```java
@RestController
@RequestMapping(value = "/api/events", produces = MediaTypes.HAL_JSON_UTF8_VALUE)
public class EventController {
    @Autowired
    private ModelMapper modelMapper;
    @Autowired
    private EventRepository eventRepository;
    @PostMapping
    public ResponseEntity create(@RequestBody EventDto eventDto) {
        Event event = modelMapper.map(eventDto, Event.class);
        Event savedEvent = eventRepository.save(event);
        URI uri = ControllerLinkBuilder.linkTo(EventController.class).slash(savedEvent.getId()).toUri();
        return ResponseEntity.created(uri).body(savedEvent);
        //linkTo로 생성된 uri를 넣어주면
        //HTTP Header에 Location=http://localhost/api/events/1 정보가 들어간다.
    }
}
```

- **EventControllerTest**

```java
public class EventControllerTest extends ControllerTest {

    @Test
    public void create() throws Exception {
        EventDto eventDto = EventDto.builder()
                .name("안녕 이벤트")
                .description("배고프다")
                .beginEnrollmentDateTime(LocalDateTime.of(2018, 11, 2, 8, 0))
                .closeEnrollmentDateTime(LocalDateTime.of(2018, 11, 3, 8, 0))
                .beginEventDateTime(LocalDateTime.of(2018, 11, 4, 8, 0))
                .endEventDateTime(LocalDateTime.of(2018, 11, 5, 8, 0))
                .basePrice(0)
                .maxPrice(0)
                .location("네이버 D2 팩토리 좁았음")
                .limitOfEnrollment(100)
                .build();

        this.mockMvc.perform(post("/api/events")
                    .contentType(MediaType.APPLICATION_JSON_UTF8)
                    .content(this.objectMapper.writeValueAsString(eventDto)))
                    .andDo(print())
                    .andExpect(status().isCreated())
                    .andExpect(header().exists("Location"))
                    .andExpect(jsonPath("free").value(false))
                    .andExpect(jsonPath("id").exists())
                    .andExpect(jsonPath("eventStatus").value(EventStatus.DRAFT.name()));
    }
}
```

> #### Slice Test
>
> 계층별 테스트를 위한 어노테이션이 존재한다. 
>
> ex) @WebMvcTest(controller만 테스트), @DataJpaTest(Jpa 관련 빈만 테스트)
>
> @DataJpaTest에서는 H2를 사용한다고 함.
>
> @WebMvcTest는 @SpringBootTest, @AutoConfigurationMockMvc 두 조합으로 대체 가능.
>
> WebMvcTest는 경량화된 테스트라 좋지만 Mocking이 너무 많이 일어나는 경우에는 @SpringBootTest를 실행하는 것이 낫다. Mocking으로 인한 버그나 장애도 발생하기 때문에 빈 생성에 오랜 시간이 걸려도 @SpringBootTest를 실행하는 것이 조금 더 정확한 테스트가 될 수 있음. 
>
> - create에서 Location 헤더 정보 설정하기
>
> POST 요청을 통해 데이터가 생성되는 경우 항상 Location 헤더 정보가 포함되어야 하며 Spring HATEOAS에서는 linkTo 메소드를 통해 해당 URI 정보를 생성해줄 수 있다.
>
> linkTo로 생성된 uri 정보를 created에 넣어주면 HTTP Header에 Location=http://localhost/api/events/1 값이 들어가게 된다.
>
> ```java
> URI uri = ControllerLinkBuilder.linkTo(EventController.class).slash(savedEvent.getId()).toUri();
>         return ResponseEntity.created(uri).body(savedEvent);
> ```
> 아래와 같은 형태로 Location 헤더 정보가 추가된다.
>
> ```json
> Headers = {Location=[http://localhost:8080/api/events/1], Content-Type=[application/hal+json;charset=UTF-8]}
> ```
## Event 생성 API에 Validation 체크 추가하기

[Event 생성 API에 Validation 체크 추가하기 커밋](https://github.com/tramyu/ksug-rest-api/commit/6457bf25839e5ac8c7320fe8fd6ae61fe2ff8781)

Validation 체크를 통해 api 요청 시 잘못된 정보가 들어왔을 때 Bad Request 반환하는 로직 추가.

@Valid 어노테이션이랑 Errors erros를 통해 해결.

min max notnull 같은 간단한건 @Valid를 통해 해결하고 그 이상의 복잡한 Validation 체크는 @Component로 생성한 Validator를 통해 해결한다.

- EventController

```java
@PostMapping
    public ResponseEntity create(@RequestBody @Valid EventDto eventDto,
                                 Errors errors) { //Errors 추가
        //추가된 부분
        if (errors.hasErrors()) {
            return ResponseEntity.badRequest().body(errors);
            //body에 Errors를 담아 리턴하지만 Java Bean 스펙이 준수되지 않아 Serialize 되지 않음.
        }

        Event event = modelMapper.map(eventDto, Event.class);
        Event savedEvent = eventRepository.save(event);
        URI uri = ControllerLinkBuilder.linkTo(EventController.class).slash(savedEvent.getId()).toUri();
        return ResponseEntity.created(uri).body(savedEvent);
    }
```

Errors 객체를 그냥 ResponseEntity에 담아서 반환해서 테스트를 하고 싶겠지만 Errors는 Java Bean 스펙이 준수된 객체가 아니라 json으로 변환해줄수가 없어서 에러가 발생한다. Custom한 Serializer를 만들어서 Jackson ObjectMapper에 등록해주어야 한다.

**@JsonComponenet로 등록한 JsonSerializer는 ObjectMapper에 자동으로 등록된다.**

- **EventErrorSerializer**

```java
@JsonComponent
public class EventErrorSerializer extends JsonSerializer<Errors> {
    @Override
    public void serialize(Errors errors, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        jsonGenerator.writeStartArray();
        errors.getFieldErrors().forEach(e -> {
            try {
                jsonGenerator.writeStartObject();
                jsonGenerator.writeStringField("field", e.getField());
                jsonGenerator.writeStringField("objectName", e.getObjectName());
                jsonGenerator.writeStringField("defaultMessage", e.getDefaultMessage());
                Object rejectedValue = e.getRejectedValue();
                if (rejectedValue != null) {
                    jsonGenerator.writeStringField("rejectedValue", rejectedValue.toString());
                } else {
                    jsonGenerator.writeStringField("rejectedValue", "");
                }
                jsonGenerator.writeEndObject();
            } catch (IOException e1) {
                e1.printStackTrace();
            }
        });

        jsonGenerator.writeEndArray();
    }
}
```

그리고 조금 더 복잡한 경우에 대한 Validation을 처리할 Validator Component를 추가하자.

- **EventDtoValidator**

```java
@Component
public class EventDtoValidator {
    public void validate(EventDto eventDto, Errors errors) {
        int maxPrice = eventDto.getMaxPrice();
        if (maxPrice < eventDto.getBasePrice()) {
            errors.rejectValue("maxPrice", "wrong.value", "Max price가 base 보다 낮으면 안되요.");
        }

        LocalDateTime closeEnrollmentDateTime = eventDto.getCloseEnrollmentDateTime();
        if (closeEnrollmentDateTime.isBefore(eventDto.getBeginEnrollmentDateTime())) {
            errors.rejectValue("closeEnrollmentDateTime", "wrong.value", "closeEnrollmentDateTime is wrong");
        }
    }
}

```

그리고 컨트롤러에 해당 Validator를 이용해 추가적인 체크를 하는 로직을 추가하자.

- **EventController**

```java
public class EventController {
    // 생략
    @Autowired
    private EventDtoValidator eventDtoValidator;
    @PostMapping
    public ResponseEntity create(@RequestBody @Valid EventDto eventDto,
                                 Errors errors) {
        // 생략
        eventDtoValidator.validate(eventDto, errors);
        if (errors.hasErrors()) {
            return ResponseEntity.badRequest().body(errors);
        }
		// 생략
    }
}

```

- **EventControllerTest**

```json
[
    {
    "field":"maxPrice",
    "objectName":"eventDto",
    "defaultMessage":"Max price가 base 보다 낮으면 안되요.",
    "rejectedValue":"500"
    },
    {
    "field":"closeEnrollmentDateTime",
    "objectName":"eventDto",
    "defaultMessage":"closeEnrollmentDateTime is wrong",
    "rejectedValue":"2018-11-04T08:00"
    }
]
```

위의 형태로 내려오는 json 값에 대한 체크와 Bad Request를 체크하는 테스트 코드 작성.

```java
@Test
public void createFailTestByCustomEventDtoValidator() throws Exception {
    EventDto eventDto = EventDto.builder()
        .name("아침에 일어나기 너무 힘듬")
        .description("진짜")
        .beginEnrollmentDateTime(LocalDateTime.of(2018, 11, 5, 8, 0))
        .closeEnrollmentDateTime(LocalDateTime.of(2018, 11, 4, 8, 0))
        .beginEventDateTime(LocalDateTime.of(2018, 11, 3, 8, 0))
        .endEventDateTime(LocalDateTime.of(2018, 11, 2, 8, 0))
        .basePrice(1000)
        .maxPrice(500)
        .location("네이버 D2 팩토리")
        .limitOfEnrollment(100)
        .build();

    this.mockMvc.perform(post("/api/events")
                         .contentType(MediaType.APPLICATION_JSON_UTF8)
                         .content(objectMapper.writeValueAsString(eventDto)))
        .andDo(print())
        .andExpect(status().isBadRequest())
        .andExpect(jsonPath("$[0].field").hasJsonPath())
        .andExpect(jsonPath("$[0].rejectedValue").hasJsonPath())
        .andExpect(jsonPath("$[0].defaultMessage").hasJsonPath())
        .andExpect(jsonPath("$[0].objectName").hasJsonPath());
}
```

이렇게 코드를 작성하면 정말로 일반적으로 만들던 REST API가 제공된다.

여기서 **Self-Descriptive와 HATEOAS를 만족하도록 link들을 추가적으로 내려주도록 변경**해야한다.

## Spring HATEOAS 적용하기

먼저 Spring Rest Docs를 먼저 적용하자. 

gradle 기준으로 작성했다.

```java
public class EventResource extends Reousrce<Event> {
    public EventResource() {
        super(contents, links);
        add(linkTo(EventController.class)).slash(~~)
    }
}
```

### Spring Rest Docs 설정 추가하기

Rest Docs를 통해 생성되는 문서를 통해서 Self-Descriptive를 제공하자.

그러기 위해서 asciidoctor 의존성을 추가하자.

- **build.gradle**

```groovy
buildscript {
    ext {
        springBootVersion = '2.1.0.RELEASE'
    }
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
        classpath 'org.asciidoctor:asciidoctor-gradle-plugin:1.5.9.2'
    }
}

apply plugin: 'java'
//생략
apply plugin: "org.asciidoctor.convert"

asciidoctor {
    doLast {
        copy {
            from file("build/asciidoc/html5")
            //src/docs/asciidoc 경로의 index.adoc 파일을 읽어서 생성된 index.html 파일 경로
            into file("src/main/resources/static/docs")
            //src/main/resources/static/docs에 위 index.html 파일을 복사한다.
        }
    }
    sourceDir = file('src/docs/asciidoc') // default path
    //위의 경로에 있는 index.adoc을 읽는다.
    options doctype: 'book', backend: 'html'
}

build {
    dependsOn asciidoctor
}

dependencies {
    // 생략
    asciidoctor 'org.springframework.restdocs:spring-restdocs-asciidoctor:2.0.2.RELEASE'
}
```

그리고 src/docs/asciidoc 경로에 템플릿 파일을 추가하자. 원래 operation 명령어가 작동해야 하는데 include만 동작하고 gradle에서 operation은 작동하지 않는다. 수정이 필요할 듯

- **index.adoc**

```json
ifndef::snippets[]
:snippets: ../../../build/generated-snippets
endif::[]
= Natural REST API Guide
백기선님 세미나;
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 4
:sectlinks:
:operation-curl-request-title: Example request
:operation-http-response-title: Example response

[[overview]]
= 개요

[[overview-http-verbs]]
== HTTP 동사

본 REST API에서 사용하는 HTTP 동사(verbs)는 가능한한 표준 HTTP와 REST 규약을 따릅니다.

|===
| 동사 | 용례

| `GET`
| 리소스를 가져올 때 사용

| `POST`
| 새 리소스를 만들 때 사용

| `PUT`
| 기존 리소스를 수정할 때 사용

| `PATCH`
| 기존 리소스의 일부를 수정할 때 사용

| `DELETE`
| 기존 리소스를 삭제할 떄 사용
|===

[[overview-http-status-codes]]
== HTTP 상태 코드

본 REST API에서 사용하는 HTTP 상태 코드는 가능한한 표준 HTTP와 REST 규약을 따릅니다.

|===
| 상태 코드 | 용례

| `200 OK`
| 요청을 성공적으로 처리함

| `201 Created`
| 새 리소스를 성공적으로 생성함. 응답의 `Location` 헤더에 해당 리소스의 URI가 담겨있다.

| `204 No Content`
| 기존 리소스를 성공적으로 수정함.

| `400 Bad Request`
| 잘못된 요청을 보낸 경우. 응답 본문에 더 오류에 대한 정보가 담겨있다.

| `404 Not Found`
| 요청한 리소스가 없음.
|===

[[overview-errors]]
== 오류

에러 응답이 발생했을 때 (상태 코드 >= 400), 본문에 해당 문제를 기술한 JSON 객체가 담겨있다. 에러 객체는 다음의 구조를 따른다.

include::{snippets}/errors/response-fields.adoc[]

예를 들어, 잘못된 요청으로 이벤트를 만들려고 했을 때 다음과 같은 `400 Bad Request` 응답을 받는다.

include::{snippets}/errors/http-response.adoc[]

[[overview-hypermedia]]
== 하이퍼미디어

본 REST API는 하이퍼미디어와 사용하며 응답에 담겨있는 리소스는 다른 리소스에 대한 링크를 가지고 있다.
응답은 http://stateless.co/hal_specification.html[Hypertext Application from resource to resource. Language (HAL)] 형식을 따른다.
링크는 `_links`라는 키로 제공한다. 본 API의 사용자(클라이언트)는 URI를 직접 생성하지 않아야 하며, 리소스에서 제공하는 링크를 사용해야 한다.

[[resources]]
= 리소스

[[resources-index]]
== 인덱스

인덱스는 서비스 진입점을 제공한다.


[[resources-index-access]]
=== 인덱스 조회

`GET` 요청을 사용하여 인덱스에 접근할 수 있다.

operation::index[snippets='response-body,http-response,links']

[[resources-events]]
== 이벤트

이벤트 리소스는 이벤트를 만들거나 조회할 때 사용한다.

[[resources-events-list]]
=== 이벤트 목록 조회

`GET` 요청을 사용하여 서비스의 모든 이벤트를 조회할 수 있다.

operation::get-events[snippets='response-fields,curl-request,http-response,links']

[[resources-events-create]]
=== 이벤트 생성

`POST` 요청을 사용해서 새 이벤트를 만들 수 있다.

:operation::create-event[snippets='response-fields,curl-request,http-response,links']

[[resources-events-get]]
=== 이벤트 조회

`Get` 요청을 사용해서 기존 이벤트 하나를 조회할 수 있다.

operation::get-event[snippets='request-fields,curl-request,http-response,links']

[[resources-events-update]]
=== 이벤트 수정

`PUT` 요청을 사용해서 기존 이벤트를 수정할 수 있다.

operation::update-event[snippets='request-fields,curl-request,http-response,links']

include::{snippets}/create-event/curl-request.adoc[]
```

기존에 작성한 테스트 코드에 아래 설정을 넣으면 Rest Docs가 제공되는데 아래처럼 설정하면 Spring에서 mockMvc를 만들어 줄 때 들어가는 설정이 누락 될 수 있다.

```java
@Before
public void setUp() {
    this.mockMvc = MockMvcBuilders.webAppContextSetup(this.context)
        		.apply(documentationConfiguration(this.restDocumentation))
        		.build();
}

//위처럼 설정하지 말고 @AutoConfigurationRestDocs를 사용하자.
```

아래처럼 @AutoConfigurationRestDocs 어노테이션을 추가하면 자동으로 설정을 해주고 기존 mockMvc를 이용한 테스트 코드에 아래 내용을 넣어주면 restdocs이 생성된다. build/generated-snippets 경로에 지정해 둔 create-event 이름으로 adoc 형식의 파일들이 생성된다.

```java
@AutoConfigurationRestDocs
public class Test {
    this.mockMvc.perform(post("/api/events")    
	.andDo(document("create-event",
    	requestFields(
            fieldWithPath("name").description("name of the event"),
            fieldWithPath("description").description("description of the event")
    	),
    	responseFields(
            fieldWithPath("id").description("identifier of the event"),
            fieldWithPath("name").description("name of the event")
    	)
	));
}
```

./gradlew clean build를 하면 아래와 같은 형태로 asciidoc이 생성된다.

![image-20181111234708775](/images/spring/image-20181111234708775.png)

위처럼 기본 설정을 넣고 만들어진 adoc 파일을 살펴보면 아래처럼 줄바꿈이 예쁘게 들어가있지 않다.

```json
[source,bash]
----
$ curl 'http://localhost:8080/api/events' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d '{"name":"안녕 이벤트","description":"배고프다","beginEnrollmentDateTime":"2018-11-02T08:00:00","closeEnrollmentDateTime":"2018-11-03T08:00:00","beginEventDateTime":"2018-11-04T08:00:00","endEventDateTime":"2018-11-05T08:00:00","location":"네이버 D2 팩토리 좁았음","basePrice":0,"maxPrice":0,"limitOfEnrollment":100}'
----
```

정렬을 이쁘게 해주려면 RestDocsMockMvcConfigurationCustomizer 빈을 등록하고 정의된 기본 테스트에서 @Import로 해당 빈을 등록해주자.

```java
import org.springframework.boot.test.autoconfigure.restdocs.RestDocsMockMvcConfigurationCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.restdocs.mockmvc.MockMvcRestDocumentationConfigurer;

import static org.springframework.restdocs.operation.preprocess.Preprocessors.prettyPrint;

@Configuration
public class RestDocIndentConfig {

    @Bean
    public RestDocsMockMvcConfigurationCustomizer customizer() {
        return new RestDocsMockMvcConfigurationCustomizer() {
            @Override
            public void customize(MockMvcRestDocumentationConfigurer configurer) {
                configurer.operationPreprocessors()
                        .withRequestDefaults(prettyPrint())
                        .withResponseDefaults(prettyPrint());
            }
        };
    }
}

//기본 어노테이션 생략
@Import(TestDocConfig.class)
public class ControllerTest {
    
}
```

위 설정을 적용하고 나면 아래처럼 정렬되어 asciidoc이 생성된다.

```json
[source,bash]
----
$ curl 'http://localhost:8080/api/events' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d '{
  "name" : "안녕 이벤트",
  "description" : "배고프다",
  "beginEnrollmentDateTime" : "2018-11-02T08:00:00",
  "closeEnrollmentDateTime" : "2018-11-03T08:00:00",
  "beginEventDateTime" : "2018-11-04T08:00:00",
  "endEventDateTime" : "2018-11-05T08:00:00",
  "location" : "네이버 D2 팩토리 좁았음",
  "basePrice" : 0,
  "maxPrice" : 0,
  "limitOfEnrollment" : 100
}'
----
```

Spring Rest Docs은 문서화를 안 한 필드가 있으면 테스트를 실패시켜 버린다. relaxed를 통해 통과시킬 수 있지만 사용하지 않는것을 권장

### HATEOAS를 만족하도록 Link 정보 제공하기

[HATEOAS를 만족하도록 Link 정보 제공하기 커밋](https://github.com/tramyu/ksug-rest-api/commit/0291a90cc319812db8dc148863532b5c0dbfafb4)

현재 우리가 내려주는 Api 정보에는 데이터만이 포함되어 있고, 여기에 Link 정보를 추가해서 HATEOAS를 만족하도록 구성해야 한다. Spring에서는 Resource 인터페이스를 제공하는데 이는 Link와 Data를 포함하고 있다. 제네릭을 이용해 Event의 Link와 Data를 담아줄 수 있는 Resource 인터페이스를 구현하자.

- **EventResource**

```java
public class EventResource extends Resource<Event> {
    public EventResource(Event content, Link... links) {
        super(content, links);
        add(linkTo(EventController.class).slash(content.getId()).withSelfRel());
    }
}
```

이제 구현된 EventResource를 컨트롤러에서 내려주도록 컨트롤러의 코드를 수정하자.

- **EventController**

```java
@PostMapping
    public ResponseEntity create(@RequestBody @Valid EventDto eventDto,
                                 Errors errors) {
		//생략
        //HATEOAS를 만족하기 위해서 Link, Profile 정보 제공
        EventResource eventResource = new EventResource(savedEvent);
        eventResource.add(linkTo(EventController.class).withRel("events"));
   eventResource.add(linkTo(EventController.class).slash(savedEvent.getId()).withRel("update"));
        eventResource.add(new Link("/docs/index.html#resources-events-create", "profile"));

        return ResponseEntity.created(uri).body(eventResource);
    }
```

위처럼 넣고 create 테스트를 돌려보면 아래와 같은 형태로 Link 정보가 들어간다.

```json
{
    "id":1,
    "name":"안녕 이벤트",
    "description":"배고프다",
    "beginEnrollmentDateTime":"2018-11-02T08:00:00",
    "closeEnrollmentDateTime":"2018-11-03T08:00:00",
    "beginEventDateTime":"2018-11-04T08:00:00",
    "endEventDateTime":"2018-11-05T08:00:00",
    "location":"네이버 D2 팩토리 좁았음",
    "basePrice":0,
    "maxPrice":0,
    "limitOfEnrollment":100,
    "offline":false,
    "free":true,
    "eventStatus":"DRAFT",
    "_links":{
        "self":{
        	"href":"http://localhost:8080/api/events/1"
        },
        "events":{
        	"href":"http://localhost:8080/api/events"
        },
        "update":{
        	"href":"http://localhost:8080/api/events/1"
        },
        "profile":{
        	"href":"/docs/index.html#resources-events-create"
        }
    }
}
```

이제 HATEOAS 조건을 만족하게 됐다. profile에 정보를 넣어서 Self-Descriptive도 만족한다.

클라이언트 개발자는 직접 하드코딩으로 update 할 주소를 입력하지 않고 json의 _links 정보를 읽어 다음 요청을 보낼 수 있다.

그리고 추가된 Link 정보가 들어있는지 테스트하는 코드도 작성하자.

- **EventControllerTest**

_links라는 json 값이 있는지, 우리가 EventResource에 넣어준 값이 있는지에 대한 테스트하고 Rest Docs에 넣어줄 links 정보를 추가하자.

```java
this.mockMvc.perform(post("/api/events")
  			//생략
            .andExpect(jsonPath("_links").hasJsonPath())
            .andExpect(jsonPath("_links.self").hasJsonPath())
            .andExpect(jsonPath("_links.events").hasJsonPath())
            .andExpect(jsonPath("_links.update").hasJsonPath())
            .andExpect(jsonPath("_links.profile").hasJsonPath())
            .andDo(document("create-event",
                links(
                    linkWithRel("self").description("link to self"),
                    linkWithRel("events").description("link to events"),
                    linkWithRel("update").description("link to update"),
                    linkWithRel("profile").description("link to profile")
                ),
                //생략
            ));
```

기존에 없던 links를 doc에 추가하면 links.adoc 파일이 추가된다.

```json
|===
|Relation|Description

|`+self+`
|link to self

|`+events+`
|link to events

|`+update+`
|link to update

|`+profile+`
|link to profile

|===
```

## 테스트, 프로덕션 환경 분리하기

[Mysql 설정 추가, Test 환경 분리 커밋](https://github.com/tramyu/ksug-rest-api/commit/1765c46683d01b0303d9f99edc59e7bb71dd0d6c)

테스트를 할 때는 경량화된 H2를 쓰는것이 좋은데 테스트 환경과 서버 실행시의 환경을 다르게 구성해보자.

- **mysql-docker.yml**

```yaml
version: '2'
services:
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: example
      MYSQL_USER: local
      MYSQL_PASSWORD: local
    ports:
      - "3306:3306"
    volumes:
      - /tmp/mysql/my.cnf:/etc/mysql/my.cnf
```
/tmp/mysql/my.cnf 경로에 utf8 설정 추가

```bash
[mysqld]
collation-server = utf8_unicode_ci
character-set-server = utf8
skip-character-set-client-handshake
```

아래 명령어로 mysql 컨테이너 실행

```bash
$ docker-compose -f mysql-docker.yml up -d
```

- **application.properties**

```properties
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect

spring.datasource.url=jdbc:mysql://localhost:3306/example?autoReconnect=true&useSSL=false&useUnicode=true&characterEncoding=utf8&serverTimezone=UTC
spring.datasource.username=local
spring.datasource.password=local
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver

logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE 
```

- **EventRepositoryTest**

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@DataJpaTest
public class EventRepositoryTest {

    @Autowired
    private EventRepository eventRepository;

    @Test
    public void crudTest() {
        Event event = Event.builder()
                    .name("Hello")
                    .description("Test")
                    .build();
        Event savedEvent = eventRepository.save(event);
        assertThat(savedEvent.getId()).isNotNull();

        List<Event> all = eventRepository.findAll();
        assertThat(all.size()).isEqualTo(1);
    }
}

```

돌려보면 잘 돌아간다. 근데 MySQL을 띄웠는데도 H2가 올라간다.

Slice Test인 **@DataJpaTest를 사용하면 H2**를 쓰도록 변경한다. Dialect를 H2를 사용하게 한다고 함.

Controller 테스트를 돌려보면 아래처럼 MySQL이 뜨는데 굳이 테스트에 무거운 DB를 띄우지 말고 H2를 쓰도록 변경하자.

```
2018-11-12 01:03:59.703  INFO 24260 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate Core {5.3.7.Final}
2018-11-12 01:03:59.705  INFO 24260 --- [           main] org.hibernate.cfg.Environment            : HHH000206: hibernate.properties not found
2018-11-12 01:03:59.847  INFO 24260 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.0.4.Final}
2018-11-12 01:03:59.976  INFO 24260 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.MySQL5Dialect
```

- **test/resources/application-test.properties**

@SpringBootTest는 어플리케이션 전체 테스트라 application.properties에 영향을 받는다.

test 경로 밑에 application-test.properties 를 생성하고 아래처럼 작성 하면 test properties의 우선순위가 더 높아져 H2를 사용하게 된다.

```properties
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.hikari.jdbc-url=jdbc:h2:mem:testdb
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect
```

그리고 테스트에 @ActiveProfile("test")를 달자.

```java
@ActiveProfiles("test")
public class ControllerTest {
}
```

그럼 아래처럼 H2가 올라오는 걸 확인할 수 있다.

```
2018-11-12 01:04:49.446  INFO 24266 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate Core {5.3.7.Final}
2018-11-12 01:04:49.447  INFO 24266 --- [           main] org.hibernate.cfg.Environment            : HHH000206: hibernate.properties not found
2018-11-12 01:04:49.589  INFO 24266 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.0.4.Final}
2018-11-12 01:04:49.720  INFO 24266 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
```

## Error에 Link 정보 내려주기

[Error에 Link 정보 내려주기 커밋](https://github.com/tramyu/ksug-rest-api/commit/63edd656ab14d36c2a5f7b7569fb0bc31509d09a)

기존에 Event 객체는 EventResource를 내려주게 변경했는데 Error에 대한 처리가 빠져있다.

ErrorResource에서 **인덱스로 가는 링크를 제공**해야 하고 따로 에러 응답에 관한 내용도 document("error", snippertes) 형태로 문서화를 해주어야 한다. 원래라면 테스트 코드를 수정해서 문서화해야하지만 귀찮아서 생략

- **EventResource**

```java
public class ErrorResource extends Resource<Errors> {
    public ErrorResource(Errors content, Link... links) {
        super(content, links);
        add(linkTo(IndexController.class).withRel("index"));
    }
}

```

- **EventController**

```java
@PostMapping
public ResponseEntity create(@RequestBody @Valid EventDto eventDto,
                             Errors errors) {
    if (errors.hasErrors()) {
        return ResponseEntity.badRequest().body(new ErrorResource(errors));
		//ErrorResource를 내려주도록 변경
    }

    eventDtoValidator.validate(eventDto, errors);
    if (errors.hasErrors()) {
        return ResponseEntity.badRequest().body(new ErrorResource(errors));
        //ErrorResource를 내려주도록 변경
    }
}
```

그리고 인덱스 페이지를 만들자. 인덱스 페이지는 다른 모든 리소스에 대한 링크들을 제공해야 한다.

지금은 Event 리소스만 존재하기 때문에 아래처럼 하나의 링크만 제공하면 된다.

- **IndexController**

```java
@RestController
public class IndexController {
    @GetMapping("/api")
    public ResourceSupport root() {
        ResourceSupport index = new ResourceSupport();
        index.add(linkTo(EventController.class).withRel("events"));
        return index;
    }
}
```

위의 컨트롤러를 추가하면 별건 없고 아래처럼 리소스에 대한 Link가 제공된다.

```json
{
    "_links":{
        "events":{
        	"href":"http://localhost:8080/api/events"
    	}
    }
}

```

## Event API CRUD, Paging 기능 추가

[Event API CRUD, Paging 기능 추가 커밋](https://github.com/tramyu/ksug-rest-api/commit/a67e07bede90690884021f9eaf56093483fa60d0)

Event 목록이 늘어나는 경우.. 한번에 다 받아올 수 없을거고 일반적으로 모든 데이터를 가져올 땐 페이징 처리를 통해 가져오는데 스프링에서는 Pageable 인터페이스를 통해 페이징을 간단하게 처리할 수 있다. 정렬도 지원하는데 여기선 사용하지 않는다.

```java
// 넘겨주는 정보는 
public ResponseEntity getEvents(Pageable pageable, PagedResourcesAssembler<Event> assembler) {
        Page<Event> page = this.eventRepository.findAll(pageable);
}

```

그리고 HATEOAS를 만족시키기 위해서 30개 중 2번째 페이지를 조회하는 경우 이전 페이지의 링크, 다음 페이지의 링크 모두를 가지고 있어야 하는데 이는 Spring의 PagedResources로 처리할 수 있다.

먼저 테스트 코드 작성, Pageable은 페이징 사이즈와 페이지 번호를 파라미터로 넘겨줘야 한다. 이것도 원래라면 document를 통해 문서를 제공해야겠지만 생략

- **EventControllerTest**

```java
@Test
public void getEvents() throws Exception {
    // Given
    IntStream.range(0, 30).forEach(this::saveEvent);
    // When & Then
    this.mockMvc.perform(get("/api/events")
                         .param("size", "10")
                         .param("page", "1"))
        .andDo(print())
        .andExpect(status().isOk())
        .andExpect(jsonPath("_links").hasJsonPath());
}

private Event saveEvent(int index) {
    Event event = Event.builder()
        .name("test event" + index)
        .build();
    return this.eventRepository.save(event);
}
```

그리고 api를 작성하자. Pageable을 파라미터로 받고 PagedResourcesAssembler를 통해서 Page 객체를 HATEOAS를 만족하는 PagedResources로 변경해서 리턴해주자.

- **EventController**

```java
@RestController
// Response type MediaTypes.HAL_JSON_UTF8_VALUE
@RequestMapping(value = "/api/events", produces = MediaTypes.HAL_JSON_UTF8_VALUE)
public class EventController {
	// 생략
    @GetMapping
    public ResponseEntity getEvents(Pageable pageable, PagedResourcesAssembler<Event> assembler) {
        Page<Event> page = this.eventRepository.findAll(pageable);
        PagedResources<EventResource> pagedResources = assembler.toResource(page, e -> new EventResource(e));
        return ResponseEntity.ok(pagedResources);
    }
}
```

위 코드를 작성하고 테스트를 돌려서 데이터가 어떤 식으로 나오는지 보면 아래와 같은 형태를 띈다. 각각 event에 self 링크를 가지고 처음 페이지, 마지막, 다음, 이전 페이지에 대한 링크 정보를 가지고 페이징에 대한 정보도 가지고 있다.

```java
{
    "_embedded":{
        "eventList":
        [
            {
                "id":11,
                "name":"test event10",
                "description":null,
                "beginEnrollmentDateTime":null,
                "closeEnrollmentDateTime":null,
                "beginEventDateTime":null,
                "endEventDateTime":null,
                "location":null,
                "basePrice":0,
                "maxPrice":0,
                "limitOfEnrollment":0,
                "offline":false,
                "free":false,
                "eventStatus":null,
                "_links":{
                    "self":{
                        "href":"http://localhost:8080/api/events/11"
                    }
                }
            },
            {
                "id":20,
                "name":"test event19",
                "description":null,
                "beginEnrollmentDateTime":null,
                "closeEnrollmentDateTime":null,
                "beginEventDateTime":null,
                "endEventDateTime":null,
                "location":null,
                "basePrice":0,
                "maxPrice":0,
                "limitOfEnrollment":0,
                "offline":false,
                "free":false,
                "eventStatus":null,
                "_links":{
                    "self":{
                        "href":"http://localhost:8080/api/events/20"
                    }
                }
            }
    	]
    },
    "_links":{
        "first":{
            "href":"http://localhost:8080/api/events?page=0&size=10"
        },
        "prev":{
            "href":"http://localhost:8080/api/events?page=0&size=10"
        },
        "self":{
            "href":"http://localhost:8080/api/events?page=1&size=10"
        },
        "next":{
            "href":"http://localhost:8080/api/events?page=2&size=10"
        },
        "last":{
            "href":"http://localhost:8080/api/events?page=2&size=10"
        }
    },
    "page":{
        "size":10,
        "totalElements":30,
        "totalPages":3,
        "number":1
    }
}
```

## Event API CRUD, 단일 이벤트 조회, 업데이트 기능 추가

[Event API CRUD, 단일 이벤트 조회, 업데이트 기능 추가 커밋](https://github.com/tramyu/ksug-rest-api/commit/db0578bc03a4bcc82f28d545b98921c58ef1d287)

이벤트 id로 하나의 이벤트 조회 기능 구현.

- **EventControllerTest**

```java
@Test
public void getEvent404() throws Exception {
    this.mockMvc.perform(get("/api/events/1983"))
        .andExpect(status().isNotFound());
}

@Test
public void getEvent() throws Exception {
    Event event = this.saveEvent(100);

    this.mockMvc.perform(get("/api/events/" + event.getId()))
        .andDo(print())
        .andExpect(status().isOk())
        .andExpect(jsonPath("name").hasJsonPath());
}
```

조회 api 별 내용은 없고 없는 경우 예외처리와 EventResource를 리턴해주는 내용.

- **EventController**

```java
@RequestMapping(value = "/api/events", produces = MediaTypes.HAL_JSON_UTF8_VALUE)
public class EventController {
    // 생략
    @GetMapping("/{id}")
    public ResponseEntity getEvent(@PathVariable Long id) {
        Optional<Event> byId = this.eventRepository.findById(id);
        if (!byId.isPresent()) {
            return ResponseEntity.notFound().build();
        }
        Event event = byId.get();
        EventResource eventResource = new EventResource(event);
        return ResponseEntity.ok(eventResource);
    }
}
```

그리고 겸사겸사 Update api 구현 생성 부분과 크게 다르지 않다.

수정하려는 이벤트가 없으면 404, 입력 데이터가 이상한 경우 400, 데이터 검증이 실패해도 400, 정상적으로 수정되는 경우 200을 반환하면 된다.

- **EventController**

```java
@RequestMapping(value = "/api/events", produces = MediaTypes.HAL_JSON_UTF8_VALUE)
public class EventController {   
    // 생략
	@PutMapping("/{id}")
    public ResponseEntity updateEvent(@PathVariable Long id,
                                      @RequestBody @Valid EventDto eventDto,
                                      Errors errors) {
        if (errors.hasErrors()) {
            return ResponseEntity.badRequest().body(new ErrorResource(errors));
        }
        eventDtoValidator.validate(eventDto, errors);
        if (errors.hasErrors()) {
            return ResponseEntity.badRequest().body(new ErrorResource(errors));
        }
        Optional<Event> byId = this.eventRepository.findById(id);
        if (!byId.isPresent()) {
            return ResponseEntity.notFound().build();
        }
        Event existingEvent = byId.get();
        modelMapper.map(eventDto, existingEvent);
        Event updatedEvent = this.eventRepository.save(existingEvent);
        return ResponseEntity.ok(new EventResource(updatedEvent));
    }
}
```



## Spring Security 적용하기

지금의 api는 권한이 없는 경우에도 모든 링크를 내려준다.

Security를 적용하고 해당 유저에게 권한이 있는 api만 보여줄 수 있도록 수정해야한다.

복잡하기도 하고 HATEOAS랑 Self-Descriptive등의 개념을 앞에서 다 다뤄서 생략.