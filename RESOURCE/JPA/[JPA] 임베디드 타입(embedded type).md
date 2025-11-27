# 사용 방법
- `@Embeddable`: 값 타입을 정의하는 곳에 표시
- `@Embedded`: 값 타입을 사용하는 곳에 표시
임베디드 타입을 포함한 모든 값 타입은 엔티티의 생명주기에 의존하므로 엔티티와 임베디드 타입 관계를 UML로 표현하면 **컴포지션(composition) 관계**가 됩니다.
> 하이버네이트는 임베디드 타입을 컴포넌트(components)라 합니다.
## 장점
- 재사용
- 높은 응집도
- Period 객체의 `isWork()` 메서드처럼 해당 값 타입만 사용하는 의미있는 메서드를 만들 수 있습니다.

# 예시
## 생년월일
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

## 회원의 근무기간과 주소를 관리
```java
// 임베디드 타입 사용
@Entity
public class Member {
  
  @Id @GeneratedVAlue
  private Long id;
  private String name;
  
  @Embedded
  private Period workPeriod;	// 근무 기간
  
  @Embedded
  private Address homeAddress;	// 집 주소
}
```

```java
// 기간 임베디드 타입
@Embeddable
public class Peroid {
  
  @Temporal(TemporalType.DATE)
  Date startDate;
  @Temporal(TemporalType/Date)
  Date endDate;
  // ...
  
  public boolean isWork (Date date) {
    // .. 값 타입을 위한 메서드를 정의할 수 있다
  }
}
```

```java
@Embeddable
public class Address {
  
  @Column(name="city") // 매핑할 컬럼 정의 가능
  private String city;
  private String street;
  private String zipcode;
  // ...
}
```

# 주의사항
## 내부 객체를 사용할때 주의사항 
`@Embedded` 뿐만 아니라 내부 객체를 사용할 때  주의할 점이다.
아래와 같이 생성해버리면 여러 엔티티가 하나의 BirthDate를 참조하기에 문제가 발생할 수 있다.
```java
@Column(name = "birth")  
@Embedded  
private BirthDate birth = new BirthDate.EMPTY;
```

https://jojoldu.tistory.com/559