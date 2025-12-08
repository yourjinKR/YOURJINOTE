# 값 타입 컬렉션이란?
```java
@ElementCollection
@CollectionTable(
	name = "favorite_sports", // 테이블명
	joinColumns = @JoinColumn(name = "member_id") // 조인할 외래키
)
@Column(name = "sport_name") // 컬렉션 테이블의 컬럼명
private Set<String> favoriteSports = new HashSet<>();
```

값 타입을 엔티티가 아닌 컬렉션에 담아 사용하는 것을 말한다.
SQL에서는 여러 테이블이 서로 조인된 형태로 보여지게 된다.

- 별도의 엔티티 클래스를 만들지 않음
- JPA가 자동으로 별도 테이블을 생성하여 관리
- 기본적으로 지연 로딩 형식으로 데이터를 가져옴
- **더티체킹 불가**
## `@ElementCollection`

```java
@Embeddable
public class SkillTag {
    private String name;
    private int value;

    // constructors, getters, setters
}
```

```java
@ElementCollection
private List<SkillTag> skillTags = new ArrayList<>();
```

컬렉션 객체임을 JPA가 알 수 있게 하게 한다.  
엔티티가 아닌 값 타입, 임베디드 타입에 대한 테이블을 생성하고 1대다 관계로 다룬다.

## `@CollectionTable`
값 타입 컬렉션을 매핑할 테이블에 대한 역할을 지정하는 데 사용한다.  
테이블의 이름과 조인정보를 적어줘야 한다.

## 사용시 고려사항
더티체킹이 불가능하기에 수정 시 컬렉션 전체를 삭제 후 전체를 삽입한다.  
변경 빈도수가 잦거나 데이터의 용향이 크다면 사용을 지양하자.
### 언제 사용할까?
- 간단한 목록을 작성할때 (카테고리...)
- ENUM 또는 문자열 기반의 고정적인 값들을 다룰때
- 수정보다는 추가/삭제가 간단한 경우


# 출처
[티스토리 -JPA 공식문서 정리](https://osumaniaddict527.tistory.com/47)

[티스토리 - JPA ElementCollection, CollectionTable을 통한 값 타입 컬렉션 사용법](https://dangdangee.tistory.com/entry/JPA-ElementCollection-CollectionTable%EC%9D%84-%ED%86%B5%ED%95%9C-%EA%B0%92-%ED%83%80%EC%9E%85-%EC%BB%AC%EB%A0%89%EC%85%98-%EC%82%AC%EC%9A%A9%EB%B2%95)



