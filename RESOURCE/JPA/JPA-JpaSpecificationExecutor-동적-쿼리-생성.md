## JpaSpecificationExecutor

- Spring Data JPA에서 제공하는 인터페이스로, **JPA Criteria API**를 사용하여 동적 쿼리를 생성할 수 있게 해준다.
- 이 인터페이스를 사용하면 복잡한 검색 조건을 쉽게 정의하고, 동적으로 쿼리를 생성하여 실행할 수 있다.
- 주요 기능으로는 동적쿼리를 생성하고 검색 조건 쿼리를 작성할 때 유연하다.

## Java 사용 방법
### 엔티티 예시
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String firstName;
    private String lastName;
    private int age;

    // getters and setters
}
```

### 레포지토리 인터페이스
```java
public interface UserRepository extends 
	JpaRepository<User, Long>, JpaSpecificationExecutor<User> {
}
```

### 스펙 정의1
```java
public class UserSpecifications {
 
    public static Specification<User> hasFirstName(String firstName) {
        return (Root<User> root, CriteriaQuery<?> query, CriteriaBuilder cb) -> {
            return cb.equal(root.get("firstName"), firstName);
        };
    }
 
    public static Specification<User> hasLastName(String lastName) {
        return (Root<User> root, CriteriaQuery<?> query, CriteriaBuilder cb) -> {
            return cb.equal(root.get("lastName"), lastName);
        };
    }
 
    public static Specification<User> isOlderThan(int age) {
        return (Root<User> root, CriteriaQuery<?> query, CriteriaBuilder cb) -> {
            return cb.greaterThan(root.get("age"), age);
        };
    }
}
```

### 스펙 정의2 (해당 방식 추천)
```java
public static Specification<Member> searchWith(
	final String gitHubId, 
	final CareerLevel careerLevel,
	final JobType jobType
	) {
    return ((root, query, builder) -> {
        List<Predicate> predicates = new ArrayList<>();
        if (StringUtils.hasText(gitHubId)) {
            predicates.add(builder.like(root.get("gitHubId"), "%" + gitHubId + "%"));
        }
        if (careerLevel != null) {
            predicates.add(builder.equal(root.get("careerLevel"), careerLevel));
        }
        if (jobType != null) {
            predicates.add(builder.equal(root.get("jobType"), jobType));
        }
        return builder.and(predicates.toArray(new Predicate[0]));
    });
}
```
해당 방식은 파라미터로 받지 않은 검색 조건에 대해서는 스펙을 정의하지 않기에
편의성을 높이면서 더욱 동적으로 쿼리를 관리할 수 있다.

### 동적쿼리 실행
```java
@Service
public class UserService {
 
    @Autowired
    private UserRepository userRepository;
 
    public List<User> findUsers(String firstName, String lastName, int age) {
        Specification<User> spec = Specification.where(null);
 
        if (firstName != null) {
            spec = spec.and(UserSpecifications.hasFirstName(firstName));
        }
 
        if (lastName != null) {
            spec = spec.and(UserSpecifications.hasLastName(lastName));
        }
 
        if (age > 0) {
            spec = spec.and(UserSpecifications.isOlderThan(age));
        }
 
        return userRepository.findAll(spec);
    }
}
```


## Kotlin에서의 사용 방법
### 엔티티 예시 (Kotlin)

> Kotlin에서는 JPA 엔티티/필드가 `open`이어야 하므로 보일러플레이트를 줄이려면 **kotlin-spring / kotlin-jpa(all-open, no-arg)** 플러그인을 사용하시는 걸 권장

```kotlin
import jakarta.persistence.*

@Entity
@Table(name = "users")
class User(
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null,

    @Column(nullable = false)
    val firstName: String,

    @Column(nullable = false)
    val lastName: String,

    @Column(nullable = false)
    val age: Int
)
```

### 레포지토리 인터페이스

```kotlin
import org.springframework.data.jpa.repository.JpaRepository
import org.springframework.data.jpa.repository.JpaSpecificationExecutor

interface UserRepository :
    JpaRepository<User, Long>,
    JpaSpecificationExecutor<User>
```

### 스펙 정의 (null/빈값은 자동 무시)

> `Specification`에서 `toPredicate`가 **null**을 반환하면 해당 조건은 **무시**

```kotlin
import org.springframework.data.jpa.domain.Specification
import jakarta.persistence.criteria.Predicate

object UserSpecs {

    fun hasFirstName(firstName: String?): Specification<User>? =
        if (firstName.isNullOrBlank()) null
        else Specification<User> { root, _, cb ->
            cb.equal(root.get<String>("firstName"), firstName)
        }

    fun hasLastName(lastName: String?): Specification<User>? =
        if (lastName.isNullOrBlank()) null
        else Specification<User> { root, _, cb ->
            cb.equal(root.get<String>("lastName"), lastName)
        }

    fun isOlderThan(age: Int?): Specification<User>? =
        if (age == null || age <= 0) null
        else Specification<User> { root, _, cb ->
            cb.greaterThan(root.get<Int>("age"), age)
        }

    // 예시: 여러 키워드 중 하나라도 firstName에 포함 (OR)
    fun firstNameContainsAny(keywords: List<String>?): Specification<User>? =
        if (keywords.isNullOrEmpty()) null
        else Specification<User> { root, _, cb ->
            val likePredicates: List<Predicate> = keywords.map { kw ->
                cb.like(root.get("firstName"), "%$kw%")
            }
            cb.or(*likePredicates.toTypedArray())
        }
}
```

### 4) 동적쿼리 실행 (권장: allOf/anyOf)

> `Specification.where(...)` 대신
> `Specification.allOf(...)`로 AND 조합, `anyOf(...)`로 OR 조합을 사용

```kotlin
import org.springframework.data.domain.Pageable
import org.springframework.stereotype.Service
import org.springframework.transaction.annotation.Transactional
import org.springframework.data.jpa.domain.Specification

@Service
class UserService(
    private val userRepository: UserRepository
) {

    @Transactional(readOnly = true)
    fun findUsers(
        firstName: String?,
        lastName: String?,
        age: Int?,
        pageable: Pageable
    ) = userRepository.findAll(
        // null 스펙은 무시되므로 listOfNotNull과 allOf로 깔끔하게
        Specification.allOf(
            listOfNotNull(
                UserSpecs.hasFirstName(firstName),
                UserSpecs.hasLastName(lastName),
                UserSpecs.isOlderThan(age)
            )
        ),
        pageable
    )
}
```

### OR/AND 혼합 예시

```kotlin
val andPart = Specification.allOf(
    listOfNotNull(
        UserSpecs.hasLastName(lastName),
        UserSpecs.isOlderThan(age)
    )
)

val orPart = Specification.anyOf(
    listOfNotNull(
        UserSpecs.firstNameContainsAny(listOf("Kim", "Lee", "Park"))
    )
)

val finalSpec = Specification.allOf(listOfNotNull(andPart, orPart))
// userRepository.findAll(finalSpec)
```

### 자주 쓰는 패턴 모음

#### `IN` 조건

```kotlin
fun lastNameIn(values: Collection<String>?): Specification<User>? =
    if (values.isNullOrEmpty()) null
    else Specification<User> { root, _, cb ->
        root.get<String>("lastName").`in`(values)
    }
```

#### 날짜/숫자 범위

```kotlin
fun ageBetween(min: Int?, max: Int?): Specification<User>? =
    when {
        min != null && max != null -> Specification { root, _, cb ->
            cb.between(root.get("age"), min, max)
        }
        min != null -> Specification { root, _, cb -> cb.greaterThanOrEqualTo(root.get("age"), min) }
        max != null -> Specification { root, _, cb -> cb.lessThanOrEqualTo(root.get("age"), max) }
        else -> null
    }
```

#### 조인(Join) 예시

```kotlin
fun hasTeamName(teamName: String?): Specification<User>? =
    if (teamName.isNullOrBlank()) null
    else Specification<User> { root, _, cb ->
        // val team = root.join<User, Team>("team", JoinType.LEFT)
        val team = root.join<Any, Any>("team") // 타입 생략 가능
        cb.equal(team.get<String>("name"), teamName)
    }
```

> **주의:** fetch join은 `Specification`에서 까다롭다
> 단건 조회라면 `@EntityGraph`로 대체하거나, 조회 전용 쿼리를 분리하는 것을 고려

### 주의사항:  `where(...)` Deprecated 대응

- **이전**: `var spec = Specification.where(null).and(...).or(...)`
    
- **이후 권장**: `Specification.allOf(listOfNotNull(...))`, `Specification.anyOf(...)`  
    또는 `listOfNotNull(...).reduce(Specification<User>::and)`처럼 **코틀린 컬렉션**으로 조립
    

## 기존 정적 쿼리와 비교
### 정적쿼리(JPA문법 사용)

- 정적 쿼리는 조건이 고정된 쿼리로, 쿼리 메서드를 통해 정의된다.
- 정적쿼리 사용 방법은 단순하고 직관적이지만, 조건이 고정되어 있어 동적인 요구사항을 처리하기 어렵다.
- 모든 조합의 메서드를 만들어야 하므로, 조건이 많아지면 관리가 복잡해진다.

```java
public interface Repository extends 
	JpaRepository<Entity, Long> {
    Page<Entity> findByKindAndGenderAndDate(String kind, String gender, String date, Pageable pageable);
}
```

### 동적쿼리(JpaSpecificationExecutor 사용)

- 동적 쿼리는 조건을 런타임에 결정할 수 있으며, Specification 인터페이스를 사용하여 정의된다.
- 여러 조건을 조합하거나 조건을 선택적으로 적용할 수 있어 유연하다.
- 동적쿼리 사용 방법은 조건이 많고 다양하게 조합되어야 하는 경우에 유리하므로 코드를 재사용할 수 있어 유지보수가 용이하다.

```java
public interface Repository extends
	JpaRepository<ProtectEntity, Long>, 
	JpaSpecificationExecutor<ProtectEntity> {
}
```

### 비교

|     | 정적쿼리                                                                                                       | 동적쿼리                                                                                                                                    |
| --- | ---------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| 장점  | - **단순함**: 조건이 몇 개 없고, 조합이 많지 않을 때는 직관적으로 작성할 수 있다.  <br>- **명시성**: 어떤 쿼리가 실행되는지 명확하게 알 수 있다.              | - **유연성**: 여러 조건을 동적으로 조합할 수 있다.  <br>- **재사용성**: 조건을 정의하는 코드의 재사용성이 높다.  <br>- **유지보수 용이**: 새로운 조건이 추가되더라도 기존 코드를 크게 변경하지 않고 추가할 수 있다. |
| 단점  | - **유연성 부족**: 조건이 많아지면 메서드의 조합이 기하급수적으로 늘어날 수 있다.  <br>- **유지보수 어려움**: 조건이 추가되거나 변경될 때마다 새로운 메서드를 추가해야 한다. | - **복잡도**: 처음 사용 시 이해하고 설정하는 과정이 조금 더 복잡할 수 있다.  <br>- **추상화**: 쿼리가 추상화되어 있어, 어떤 SQL이 실행되는지 바로 알기 어려울 수 있다.                             |
| 결론  | 조건이 단순하고 고정된 경우, 메서드 명만으로 충분히 표현 가능한 경우에 적합하다.                                                             | 조건이 다양하고, 동적으로 조합해야 하는 경우, 그리고 유지보수와 확장성이 중요한 경우에 적합하다                                                                                  |
**결론 : 언제 무엇을 사용할까?**
정적쿼리 : 조건이 단순하고 메서드 명으로 충분히 표현 가능할때
동적쿼리: 조건이 다양하고 동적으로 조합할 경우, 유지보수와 확장성이 중요한 경우

# 출처
[JPA 정적쿼리, 동적쿼리(feat. JpaSpecificationExecutor)-티스토리](feat.%20JpaSpecificationExecutor)-티스토리)

[동적 쿼리 생성을 위해 JPA Specification 적용-티스토리](https://wookjongbackend.tistory.com/41)

[테코블](https://tecoble.techcourse.co.kr/post/2022-10-11-jpa-dynamic-query/)