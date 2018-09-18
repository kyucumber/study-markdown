# Java8 LocalDateTime custom Deserializer, Serializer 만들기


Spring에서 Java8의 LocalDateTime을 그냥 사용하다보면 왠진 모르겠지만 Serialize, Deserialize 하는 과정에서 많은 문제가 발생한다.

그냥 아래처럼 Custom Serializer, Deserializer 두개를 따로 만들고 활용하는게 더 좋은 것 같다. 원하는 형식에 맞게 Pattern을 지정하고 사용하면 별 에러 없이 사용할 수 있었다. 내가 LocalDateTime을 잘 몰라서 그럴수도 있고..

- **LocalDateTimeSerializer.java**

```java
import java.io.IOException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;

public class LocalDateTimeSerializer extends JsonSerializer<LocalDateTime> {
    private static final DateTimeFormatter DATE_FORMAT = DateTimeFormatter.ofPattern("yyyy.MM.dd HH:mm");
    @Override
    public void serialize(LocalDateTime value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeString(value.format(DATE_FORMAT));
    }
}
```

- **LocalDateTimeDeserializer.java**

```java
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonDeserializer;

import java.io.IOException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class LocalDateTimeDeserializer extends JsonDeserializer<LocalDateTime> {
    private static final DateTimeFormatter DATE_FORMAT = DateTimeFormatter.ofPattern("yyyy.MM.dd HH:mm");

    @Override
    public LocalDateTime deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JsonProcessingException {
        return LocalDateTime.parse(jsonParser.getText(), DATE_FORMAT);
    }
}
```

LocalDateTime에 사용할 패턴을 정해서 위처럼 Deserializer와 Serializer를 만들고 아래처럼 LocalDateTime 필드에 적용해서 사용하면 된다.

```java
@MappedSuperclass
@EntityListeners(value = {AuditingEntityListener.class})
@Getter
public abstract class AuditingDateEntity implements Serializable {

    @Column(nullable = false, updatable = false)
    @JsonSerialize(using = LocalDateTimeSerializer.class)
    @JsonDeserialize(using = LocalDateTimeDeserializer.class)
    @CreatedDate
    private LocalDateTime createdDate;
}
```



