JPA에서 데이터를 조회하는 방식에는 2가지 방식이 존재한다.

## findById()

```java
Optional<T> findById(ID id)
```
- Optional로 감싸서 반환함
- CrudRepository와 JpaRepository 인터페이스에서 모두 제공되는 메소드

## getReferenceById()
```java
public T getReferenceById(ID id)
```
- 타입 그대로 반환
- 해당 엔티티가 영속성 컨텍스트에 없다면 `EntityNotFoundException`이 발생
- JpaRepository 인터페이스에서만 제공
### 데이터 로딩 방식
- 지연 로딩으로 가져옴
- Proxy 객체로 가지고 있음



