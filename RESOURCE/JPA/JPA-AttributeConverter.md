# AttributeConverter
DB에 어떤 값을 저장할 때 전/후처리 작업을 수행

## 구조
```java
public interface AttributeConverter<X,Y> {  
     Y convertToDatabaseColumn(X attribute);  
     X convertToEntityAttribute(Y dbData);  
}
```
인터페이스를 구현하여 진행

## 사용 목적
DB에 어떤 값을 저장할 때 전/후처리가 필요한 요구사항이 있을 수 있다. 예를 들어, 사용자의 개인 정보를 암호화해서 저장하는 경우 혹은 데이터베이스에 저장된 값을 불러올 때 Static 한 인스턴스로 변환해야 하는 경우이다.

# 예시
## JSON 컨버터
### JsonMapConverter
```json
import jakarta.persistence.AttributeConverter;  
import jakarta.persistence.Converter;  
import java.util.HashMap;  
import java.util.Map;  
import lombok.RequiredArgsConstructor;  
import tools.jackson.core.JacksonException;  
import tools.jackson.databind.ObjectMapper;  
  
@Converter  
@RequiredArgsConstructor  
public class JsonMapConverter implements AttributeConverter<Map<Object, String>, String> {  
  
    private final ObjectMapper mapper = new ObjectMapper();  
  
    @Override  
    public String convertToDatabaseColumn(Map<Object, String> attribute) {  
        if (attribute == null) {return null;}  
  
        try {  
            return mapper.writeValueAsString(attribute);  
        } catch (JacksonException e) {  
            throw new RuntimeException("json 파싱 중 에러", e);  
        }  
    }  
  
    @Override  
    public Map<Object, String> convertToEntityAttribute(String dbData) {  
  
        if (dbData == null || dbData.isEmpty()) {  
            return new HashMap<>();  
        }  
  
        try {  
            return mapper.readValue(dbData, Map.class);  
        } catch (JacksonException e) {  
            throw new RuntimeException("json 파싱 중 에러", e);  
        }  
    }  
}
```
### 엔티티에 적용
```java
@Convert(converter = JsonMapConverter.class)  
@Column(name = "details", columnDefinition = "json")  
@Builder.Default  
private Map<String, Object> details = new HashMap<>();
```

# 출처
[json 타입 버전별 사용법](https://velog.io/@happyjamy/JPA-%EC%97%90%EC%84%9C-MySql-Json-%ED%83%80%EC%9E%85-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-hibernate-6-%EC%9D%B4%EC%83%81)

[AttributeConverter 설정법](https://rachel0115.tistory.com/entry/JPA-JPA-AttributeConverter%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-%EC%9E%90%EB%8F%99%EC%9C%BC%EB%A1%9C-%EA%B0%92%EC%9D%84-%EB%B3%80%ED%99%98%ED%95%98%EA%B8%B0)