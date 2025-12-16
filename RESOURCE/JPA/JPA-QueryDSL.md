## 주의사항
### interFaceName + Impl

클래스명이 "인터페이스명 + Impl" 인지 주의하자

> jpa가 해당 이름 형식으로 찾기 때문

```java
public interface ExerciseRepositoryQueryDSL {
    Page<Exercise> search(Pageable pageable, ExerciseCategory category, boolean hidden);
}

@RequiredArgsConstructor
public class ExerciseRepositoryQueryDSLImpl implements ExerciseRepositoryQueryDSL {

    private final JPAQueryFactory queryFactory;
}
```

### static import

해당 방식으로 정적 import 할 시에 더 깔끔한 코드 작성 가능

```java
import static app.fitsync.domain.exercise.entity.QExercise.exercise;
```


### 페이지네이션 유틸

페이지네이션 코드 작성은 반복적인 작업이 될 수 있기에 이와 같이 작성

```java
@Repository  
public class QueryDslRepositorySupport {  
  
    protected final JPAQueryFactory queryFactory;  
  
    public QueryDslRepositorySupport(JPAQueryFactory queryFactory) {  
        this.queryFactory = queryFactory;  
    }  
  
    public <T> Page<T> applyPagination(  
            Pageable pageable,  
            JPAQuery<T> contentQuery,  
            JPAQuery<Long> countQuery) {  
  
        List<T> content = contentQuery  
                .offset(pageable.getOffset())  
                .limit(pageable.getPageSize())  
                .fetch();  
  
        return PageableExecutionUtils.getPage(content, pageable, countQuery::fetchOne);  
    }  
}
```