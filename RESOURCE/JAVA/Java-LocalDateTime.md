# LocalDateTime

날짜와 시간을 동시에 관리하는 클래스이다.

## 프론트에서 어떻게 전달할까?

Spring(Jackson)이 **ISO-8601** 형식을 기본 지원하기에 아래와 같은 문자열로 자동 파싱된다.

```json
{
  "createdAt": "2026-01-29T12:34:56"
}
```

```java
record ExampleDTO(LocalDateTime createdAt) {
}
```

# 활용법
## 나이 계산
```java
public Long getAge() {  
	LocalDateTime now = LocalDateTime.now();  
	return ChronoUnit.YEARS.between(birth, now);  
}  
```

## JPA에서 생년월일 별개의 객체로 관리
### 생년월일을 별개의 객체로 관리
```java
import jakarta.persistence.Embeddable;  
import java.time.LocalDateTime;  
import java.time.temporal.ChronoUnit;  
import lombok.NoArgsConstructor;  
  
@Embeddable  
@NoArgsConstructor  
public class BirthDate {  
    public static BirthDate EMPTY = new BirthDate(null);  
    private LocalDateTime birth;  
  
    public BirthDate(LocalDateTime localDateTime) {  
        this.birth = localDateTime;  
    }  
  
    public Long getAge() {  
        LocalDateTime now = LocalDateTime.now();  
        return ChronoUnit.YEARS.between(birth, now);  
    }  
}
```

```java
static BirthDate toBirthDate(BirthReqeust reqeust) {  
    if (reqeust == null) { return BirthDate.EMPTY; }  
  
    LocalDateTime birth = LocalDateTime.of(  
            reqeust.year(),  
            reqeust.month(),  
            reqeust.dayOfMonth(),  
            0,  
            0  
    );  
  
    return new BirthDate(birth);  
}
```
객체는 위와 같이 생성함


