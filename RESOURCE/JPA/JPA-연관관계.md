# JPA 연관관계: @OneToMany와 @ManyToOne

## 개념 정리

### @ManyToOne

- 다대일(N:1) 관계를 매핑할 때 사용한다.
- 데이터베이스에서 `외래 키`를 가진 쪽(즉, 다수 쪽 테이블)에 선언한다.
- 기본적으로 `fetch = FetchType.EAGER`가 적용된다.
- 실무에서는 `fetch = FetchType.LAZY`로 설정하는 경우가 많다.

예시: 여러 개의 주문(Order)은 하나의 회원(Member)에 속한다.

```java
@Entity
public class Order {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)  // 다대일 매핑
    @JoinColumn(name = "member_id")     // 외래 키
    private Member member;

    private String orderDetail;

    // Getter, Setter
}
```

| 속성             | 타입              | 기본값     | 실무 팁                                              | 설명                    |
| -------------- | --------------- | ------- | ------------------------------------------------- | --------------------- |
| `fetch`        | `FetchType`     | `EAGER` | N+1 이슈를 줄이려면 `LAZY` 권장                            | 연관 엔티티 조회 전략          |
| `cascade`      | `CascadeType[]` | 없음      | N:1에서 부모 삭제로 이어지는 `REMOVE`는 주의. 대개 명시적으로 저장/삭제 처리 | 엔티티 상태 전이를 전파         |
| `optional`     | `boolean`       | `true`  | `NOT NULL` 제약이 필요하면 `false` 설정 후 DDL도 함께 관리       | 외래 키 `NULL` 허용 여부     |
| `targetEntity` | `Class`         | 유추      | 대부분 불필요                                           | 제네릭으로 타입 유추가 불가할 때 명시 |

### @OneToMany

- 일대다(1\:N) 관계를 매핑할 때 사용한다.
- 데이터베이스에서는 일대다 단방향 매핑이 존재하지 않고, 실제로는 **다대일의 반대편** 개념이다.
- `mappedBy` 속성을 통해 연관관계의 주인이 아님을 명시해야 한다.
- 기본적으로 `fetch = FetchType.LAZY`가 적용된다.

예시: 회원(Member)은 여러 개의 주문(Order)을 가진다.

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "member")   // 연관관계의 주인(Order.member)에 의해 관리됨
    private List<Order> orders = new ArrayList<>();

    // Getter, Setter
}

```

| 속성              | 타입              | 기본값     | 설명                   | 실무 팁                                           |
| --------------- | --------------- | ------- | -------------------- | ---------------------------------------------- |
| `mappedBy`      | `String`        | 없음      | 주인이 아님을 표시(반대편 필드명)  | 양방향일 때 반드시 지정하여 외래 키 업데이트를 주인에게 맡김             |
| `fetch`         | `FetchType`     | `LAZY`  | 컬렉션 지연 로딩            | 기본값 유지 권장. 필요 시 JPQL fetch join, BatchSize로 튜닝 |
| `cascade`       | `CascadeType[]` | 없음      | 컬렉션 요소에 상태 전파        | 부모 저장 시 자식 자동 저장이 필요하면 `PERSIST`/`MERGE` 사용    |
| `orphanRemoval` | `boolean`       | `false` | 컬렉션에서 제거된 엔티티를 자동 삭제 | “부모-자식 생명주기”일 때 유용. 실수로 데이터 손실되지 않게 주의         |
| `targetEntity`  | `Class`         | 유추      | 제네릭 불가 시 명시          | 대부분 불필요                                        |

## 연관관계의 주인

- JPA에서 외래 키를 가진 엔티티가 연관관계의 주인이다.
- 즉, `@ManyToOne`이 연관관계의 주인이며, `@OneToMany(mappedBy = ...)`는 주인이 아니다.
- 연관관계의 주인이 아닌 쪽은 단순히 조회(read)만 가능하다.

## 양방향 매핑 예시

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}

@Entity
public class Order {
    @Id @GeneratedValue
    private Long id;
    private String orderDetail;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;
}

```

## 정리

- `@ManyToOne`: 다수 → 단일, 외래 키를 가짐, 연관관계 주인.
    
- `@OneToMany`: 단일 → 다수, `mappedBy` 필수, 주인이 아님.
    
- 실무에서는 보통 `@ManyToOne`을 주로 사용하고, `@OneToMany`는 필요 시 조회 용도로만 사용한다.
    
- 추가 참고
    
    # JPA 연관관계 정리: `@ManyToOne` / `@OneToMany` 속성 집중
    
    백엔드 웹개발 실무에서 가장 많이 쓰는 연관관계는 N:1과 1:N입니다. 아래는 두 어노테이션의 “속성(Attributes)”을 중심으로, 기본값과 실무 팁까지 정리한 노션용 문서입니다.
    
    ## 요약
    
    - `@ManyToOne`은 외래 키를 가진 쪽(다수 쪽)에 선언되며, 기본 Fetch 전략이 `EAGER`입니다. 실무에서는 성능을 위해 대부분 `LAZY`로 바꿉니다.
    - `@OneToMany`는 보통 반대편의 읽기 전용 뷰로 사용합니다. 기본 Fetch 전략은 `LAZY`이며, `mappedBy`로 주인이 아님을 명시합니다.
    - 외래 키는 항상 N:1의 N 쪽에 있습니다. 따라서 연관관계의 주인은 대개 `@ManyToOne`입니다.
    
    ---
    
    ## `@ManyToOne` 속성 상세
    
    |속성|타입|기본값|설명|실무 팁|
    |---|---|---|---|---|
    |`fetch`|`FetchType`|`EAGER`|연관 엔티티 조회 전략|N+1 이슈를 줄이려면 `LAZY` 권장|
    |`cascade`|`CascadeType[]`|없음|엔티티 상태 전이를 전파|N:1에서 부모 삭제로 이어지는 `REMOVE`는 주의. 대개 명시적으로 저장/삭제 처리|
    |`optional`|`boolean`|`true`|외래 키 `NULL` 허용 여부|`NOT NULL` 제약이 필요하면 `false` 설정 후 DDL도 함께 관리|
    |`targetEntity`|`Class`|유추|제네릭으로 타입 유추가 불가할 때 명시|대부분 불필요|
    
    참고: `mappedBy`, `orphanRemoval`은 `@ManyToOne`에는 없습니다. `@ManyToOne` 쪽이 보통 연관관계의 주인입니다.
    
    ---
    
    ## `@OneToMany` 속성 상세
    
    |속성|타입|기본값|설명|실무 팁|
    |---|---|---|---|---|
    |`mappedBy`|`String`|없음|주인이 아님을 표시(반대편 필드명)|양방향일 때 반드시 지정하여 외래 키 업데이트를 주인에게 맡김|
    |`fetch`|`FetchType`|`LAZY`|컬렉션 지연 로딩|기본값 유지 권장. 필요 시 JPQL fetch join, BatchSize로 튜닝|
    |`cascade`|`CascadeType[]`|없음|컬렉션 요소에 상태 전파|부모 저장 시 자식 자동 저장이 필요하면 `PERSIST`/`MERGE` 사용|
    |`orphanRemoval`|`boolean`|`false`|컬렉션에서 제거된 엔티티를 자동 삭제|“부모-자식 생명주기”일 때 유용. 실수로 데이터 손실되지 않게 주의|
    |`targetEntity`|`Class`|유추|제네릭 불가 시 명시|대부분 불필요|
    
    추가로 정렬/순서를 제어하려면 `@OrderBy`(정렬만) 또는 `@OrderColumn`(순서 컬럼 유지)을 사용할 수 있습니다.
    
    ---
    
    ## 조인 관련 어노테이션 함께 이해하기
    
    대부분의 실무 혼란은 “외래 키가 어디에 있고 누가 주인인가”에서 비롯됩니다. 아래 어노테이션을 함께 이해하면 전체 이미지가 선명해집니다.
    
    ### `@JoinColumn`
    
    - 위치: 주인(외래 키를 가진 엔티티) 필드에 선언
    - 주요 속성
        - `name`: 외래 키 컬럼명. 예) `member_id`
        - `referencedColumnName`: 참조 대상 컬럼. 기본값은 PK
        - `nullable`: `NOT NULL` 제약 반영
        - `unique`: 유니크 제약
        - `insertable`, `updatable`: 읽기 전용 컬럼로 만들 때 사용(특수 케이스)
        - `columnDefinition`, `foreignKey`: DDL 커스터마이징
    - 복합키를 참조하면 `@JoinColumns`로 다중 지정
    
    ### `@JoinTable`
    
    - 표준 JPA에서 **단방향 `@OneToMany`*를 만들 때 기본 방식은 조인 테이블입니다.
    - 그러나 조인 테이블은 성능/관리 면에서 비용이 큽니다. 실무에서는 되도록 양방향 관계(`mappedBy`)로 설계하여 외래 키를 자식 테이블에 두는 편이 일반적입니다.
    
    ---
    
    ## 예시 1) 가장 흔한 양방향 N:1/1:N 매핑
    
    요구사항
    
    - 회원(Member) 1명이 여러 주문(Order)을 가진다.
    - 외래 키는 `order.member_id`에 둔다.
    - 조회는 지연 로딩, 저장은 부모에서 자식으로 전파, 고아 제거 적용.
    
    ```java
    // 부모
    @Entity
    public class Member {
    
        @Id @GeneratedValue
        private Long id;
    
        private String name;
    
        // 주인이 아님. 주인은 Order.member
        @OneToMany(
            mappedBy = "member",
            fetch = FetchType.LAZY,
            cascade = {CascadeType.PERSIST, CascadeType.MERGE},
            orphanRemoval = true
        )
        private List<Order> orders = new ArrayList<>();
    
        // 편의 메서드: 양쪽 세팅을 일관되게 유지
        public void addOrder(Order order) {
            orders.add(order);
            order.setMember(this);
        }
    
        public void removeOrder(Order order) {
            orders.remove(order);   // orphanRemoval=true 이면 DB에서 DELETE
            order.setMember(null);
        }
    
        // getters/setters
    }
    
    // 자식(연관관계의 주인)
    @Entity
    public class Order {
    
        @Id @GeneratedValue
        private Long id;
    
        private String orderNo;
    
        @ManyToOne(fetch = FetchType.LAZY, optional = false)
        @JoinColumn(name = "member_id", nullable = false)
        private Member member;
    
        // getters/setters
    }
    
    ```
    
    포인트
    
    - 외래 키는 `Order.member_id`에 존재합니다. 주인은 `Order.member`입니다.
    - `Member.addOrder(...)` 같은 편의 메서드로 양쪽 관계를 동시에 맞춰주어 일관성을 유지합니다.
    - `orphanRemoval = true` 덕분에 `Member.orders`에서 제거하면 해당 `Order`가 삭제됩니다. 도메인 요구가 “부모가 생명주기를 관리”할 때만 사용합니다.
    
    ---
    
    ## 예시 2) 단방향 `@OneToMany`가 꼭 필요할 때
    
    표준 JPA는 단방향 1:N에서 **조인 테이블**을 사용합니다.
    
    ```java
    @Entity
    public class Member {
    
        @Id @GeneratedValue
        private Long id;
    
        private String name;
    
        // 조인 테이블: member_orders(member_id, order_id)
        @OneToMany
        @JoinTable(
            name = "member_orders",
            joinColumns = @JoinColumn(name = "member_id"),
            inverseJoinColumns = @JoinColumn(name = "order_id")
        )
        private List<Order> orders = new ArrayList<>();
    }
    
    @Entity
    public class Order {
    
        @Id @GeneratedValue
        private Long id;
    
        private String orderNo;
    }
    
    ```
    
    포인트
    
    - 관리 테이블이 하나 더 생기고, 쿼리/인덱스 관리가 복잡해질 수 있습니다.
    - 가능하다면 양방향 매핑으로 전환해 외래 키를 자식 테이블에 두는 설계를 권장합니다.
    
    ---
    
    ## 설계와 성능에 대한 실무 팁
    
    1. 연관관계의 주인은 가능한 간단하게
        
        외래 키를 가진 `@ManyToOne` 쪽을 주인으로 두고, 반대편은 `mappedBy`로 읽기 전용 뷰로 둡니다.
        
    2. 기본 Fetch 전략 점검
        
        - `@ManyToOne` 기본 `EAGER`는 실무상 위험합니다. 반드시 `LAZY`로 변경을 고려하십시오.
        - 컬렉션은 기본이 `LAZY`입니다. 대량 조회 시 JPQL `fetch join`, Hibernate `@BatchSize` 등으로 N+1을 줄입니다.
    3. `cascade`는 필요한 범위만
        
        - 부모가 자식의 생명주기를 전담할 때만 `PERSIST`, `REMOVE` 등을 신중히 사용합니다.
        - `@ManyToOne`에 `REMOVE`를 주면 자식 삭제가 부모 삭제로 전파될 수 있어 대개 금지합니다.
    4. `orphanRemoval` 사용 기준
        
        - 부모와 자식이 강한 소유 관계(예: 주문과 주문항목)일 때만 사용합니다.
        - 실수로 컬렉션에서 제거했는데 실제로 `DELETE`가 나가기 때문에 테스트를 충분히 수행합니다.
    5. 편의 메서드로 양쪽 일관성 유지
        
        - `addChild/removeChild` 메서드로 두 엔티티의 양쪽 참조를 항상 함께 갱신하여, 영속성 컨텍스트와 DB 상태 불일치를 방지합니다.
    6. 제약조건을 엔티티와 DDL로 함께 관리
        
        - `optional=false`, `@JoinColumn(nullable=false)`처럼 코드와 스키마가 같은 제약을 표현하도록 맞춰야 버그가 줄어듭니다.
    
    ---
    
    ## 작은 체크리스트
    
    - `@ManyToOne(fetch = LAZY)`로 시작했는가
    - 반대편 `@OneToMany(mappedBy = "...")`로 주인이 아님을 명확히 했는가
    - 편의 메서드로 양쪽 참조를 항상 함께 설정/해제하는가
    - `cascade`, `orphanRemoval`을 도메인 생명주기에 맞게 최소한으로 설정했는가
    - 대량 조회 구간에서 N+1 튜닝 전략(fetch join, BatchSize)을 계획했는가
    
    ---
    
    ## 추가 예시: 선택적 연관관계, NULL 허용
    
    ```java
    @Entity
    public class Post {
    
        @Id @GeneratedValue
        private Long id;
    
        private String title;
    
        // 작성자를 추후에 채울 수 있게 optional=true, FK NULL 허용
        @ManyToOne(fetch = FetchType.LAZY, optional = true)
        @JoinColumn(name = "author_id", nullable = true)
        private Member author;
    }
    
    ```
    
    포인트
    
    - `optional=true`는 기본값이며, `nullable=true` DDL과 의미가 일치합니다.
    - 반드시 NULL을 허용해야 하는 도메인에서만 사용합니다.
    
    ---
    
    필요하시면 위 예시를 기반으로 테스트 코드(JPA/Hibernate)나 JPQL 쿼리 최적화 패턴(fetch join, BatchSize)까지 이어서 정리해드리겠습니다.