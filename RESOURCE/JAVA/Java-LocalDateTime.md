# LocalDateTime

날짜와 시간을 동시에 관리하는 클래스이다.



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


