# EntityGraph

## 1) EntityGraph란

`@EntityGraph`는 특정 조회 메서드에서만 연관 엔티티를 **미리 같이 로딩**하게 만드는 기능이다.  
엔티티 매핑은 `LAZY`로 유지하면서, 필요한 조회에서만 fetch 전략을 바꾼다.

## 2) 사용 목적

예를 들어 게시글(Post)과 작성자(User)가 있고, `Post -> User`가 LAZY라고 가정한다.

- 게시글 목록을 조회하는 쿼리 1번
- 목록을 돌면서 `post.getUser().getName()` 같은 접근을 할 때마다 작성자 조회 쿼리가 추가로 발생
- 결과적으로 1 + N번이 되어 **N+1 문제**가 된다.

## 3) 기본 사용법

```java
@EntityGraph(attributePaths = "user")
List<Post> findAllByOrderByIdDesc();
```

`attributePaths`에 연관 필드명을 넣으면 해당 연관을 조회 시점에 같이 로딩한다. 보통 JOIN 형태로 한 번에 가져오게 된다.

## 4) 사용 시기

- 목록/상세에서 연관 데이터를 거의 항상 바로 사용할 때
- LAZY로 인한 추가 쿼리(N+1)나 트랜잭션 밖 접근 문제를 줄이고 싶을 때
- 특히 `ManyToOne`, `OneToOne` 같은 ToOne 연관에서 효과가 좋다.
    
## 5) 주의사항

- `OneToMany`, `ManyToMany` 같은 ToMany 연관을 같이 땡기면 JOIN 결과가 뻥튀기될 수 있다.
- 페이징이 있는 목록에서 ToMany fetch는 성능/정확성 이슈가 생기기 쉽다.

## 6) DTO 조회와의 선택 기준

목록 응답이 DTO라면 DTO 프로젝션이 더 효율적인 경우가 많다.

- DTO 프로젝션: 필요한 컬럼만 SELECT, N+1 구조 자체가 없음
- EntityGraph: 엔티티를 가져오면서 특정 연관만 미리 로딩, 재사용/도메인 로직에 유리