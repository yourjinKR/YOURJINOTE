기존에 사용하던 `BirthDate`는 `LocalDateTime` 타입을 사용함.
그 이유는 프로젝트에서 모든 시간 타입은 해당 타입으로 통일했기 때문에.

```java
package app.fitsync.domain.user.entity;  
  
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
  
        if (this.birth == null) throw new IllegalArgumentException("생년월일 정보가 없습니다");  
  
        return ChronoUnit.YEARS.between(birth, now);  
    }  
}
```

그러나 생년월일 계산 캡슐화를 위해서 해당 데이터를 별도의 객체로 분리한 만큼 타입 또한 더욱 핏하게 다뤄야 할 필요성을 느낌.

즉, 도메인의 목적을 고려하여 변경함.

```java
package app.fitsync.domain.user.entity;  
  
import jakarta.persistence.Embeddable;  
import java.time.LocalDate;  
import java.time.LocalDateTime;  
import java.time.temporal.ChronoUnit;  
import lombok.NoArgsConstructor;  
  
@Embeddable  
@NoArgsConstructor  
public class BirthDate {  
    private LocalDate birth;  
  
    public BirthDate(LocalDate localDate) {  
        this.birth = localDate;  
    }  
  
    public BirthDate(LocalDateTime localDateTime) {  
        this.birth = (localDateTime != null) ? localDateTime.toLocalDate() : null;  
    }  
  
    public Long getAge(LocalDate now ) {  
        if (this.birth == null) throw new IllegalArgumentException("생년월일 정보가 없습니다");  
  
        return ChronoUnit.YEARS.between(birth, now);  
    }  
}
```