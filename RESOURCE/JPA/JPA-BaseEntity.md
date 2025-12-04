# BaseEntity
```java
import jakarta.persistence.Column;  
import jakarta.persistence.EntityListeners;  
import jakarta.persistence.MappedSuperclass;  
import java.time.LocalDateTime;  
import lombok.Getter;  
import org.springframework.data.annotation.CreatedDate;  
import org.springframework.data.annotation.LastModifiedDate;  
import org.springframework.data.jpa.domain.support.AuditingEntityListener;  
  
@Getter  
@MappedSuperclass  
@EntityListeners(AuditingEntityListener.class)  
public abstract class BaseEntity {  
  
    @CreatedDate  
    @Column(name = "created_at", nullable = true, updatable = false)  
    private LocalDateTime createdAt;  
  
    @LastModifiedDate  
    @Column(name = "updated_at")  
    private LocalDateTime updatedAt;  
  
    @Column(name = "deleted_at", nullable = true)  
    private LocalDateTime deletedAt;  
}
```

## @MappedSuperclass
- 상속 허락 어노테이션
- 해당 어노테이션이 적용된 클래스는 테이블로 생성되지 않음
- 상속하는 하위 클래스들이 BaseEntity의 필드와 매핑된 테이블을 만든다
- 엔티티 클래스는 엔티티 클래스끼리만 상속할 수 있지만 해당 어노테이션으로 상속할 수 있게 해줌
## @EntityListeners(AuditingEntityListener.classPrePersit
`PreUpdate`등 Entity에 LifeCytcle과 관련된 이벤트들이 존재하는데 해당 이벤트를 Listen해주는것이다.
### PrePersis
순수 JPA는 EntityManager를 이용해 persist() 메소드를 통해 영속화 작업을 한다.  
그렇기 때문에, persist() 메서드가 호출되기 전, prePersist() 메소드가 실행되는데, 해당 메소드 안에서 작동하는 로직을 나타내기 위해 작성하고, 이 안에서 created_at, updated_at을 현재 시각으로 수정시켜준다.
### PreUpdate
변경 감지를 통해 update쿼리가 실행 되었을 때 해당 어노테이션에서 preUpdate()메서드를 통해 updated_at의 시간을 현재시각으로 수정시켜준다.

## 추상 클래스로 사용하는 이유
공통기능에 대한 확장이기 때문에
[[Java-추상-클래스와-인터페이스-비교]]
## @CreatedDate
PrePersist와 비슷한 기능으로, Entity가 생성되어 저장될 때 시간이 자동으로 저장이 된다.
## @LastModifiedDate
PreUpdate와 비슷한 기능으로, 조회한 Entity의 값을 변경할 때 시간이 자동으로 저장된다.

# 출처
[출처1](https://velog.io/@vpdls1511/Spring%EC%97%90%EC%84%9C-BaseEntity%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0)
[출처2](https://passionfruit200.tistory.com/389)

