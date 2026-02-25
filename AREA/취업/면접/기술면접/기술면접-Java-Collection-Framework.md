# 다시 점검할 부분


## 1. Map 구현체 선택과 equals / hashCode 계약

Spring과 실무에서 **가장 많이 사고가 나는 지점**입니다.

### 지뢰 질문

- `HashMap`의 key로 DTO 객체를 사용했는데 조회가 되지 않는다면, 가장 먼저 의심해야 할 것은 무엇인가?
    
- Spring에서 `@Cacheable`의 key가 객체일 때, equals / hashCode 구현이 왜 캐시 미스의 원인이 될 수 있을까?
    
- `HashMap`에 key를 넣은 뒤, key 객체의 필드 값을 변경하면 어떤 일이 발생하며, 이는 컴파일 타임과 런타임 중 언제 드러나는가?
    
- `ConcurrentHashMap`은 thread-safe한데도 불구하고, key 객체의 불변성이 중요한 이유는 무엇인가?
    

### 핵심 연결

- equals / hashCode 계약
    
- Hash 기반 컬렉션 동작 원리
    
- Spring Cache, 세션, 컨텍스트 내부 Map 사용

## 2. List vs Set 선택이 Spring 동작에 미치는 영향

컬렉션 선택이 **중복 처리, 순서, 의도 표현**에 직접적인 영향을 줍니다.

### 지뢰 질문

- Spring에서 `@Autowired List<BeanType>`과 `Set<BeanType>`을 주입받을 때, 어떤 차이가 발생할 수 있는가?
    
- 중복 Bean이 존재할 때 `List`는 허용되는데 `Set`에서는 제거되는 이유는 무엇이며, 이는 어떤 메서드 구현에 의존하는가?
    
- `Set`을 사용했는데 중복이 제거되지 않았다면, 가장 먼저 점검해야 할 클래스 구현은 무엇인가?
    
- Bean 주입 순서가 중요한 경우 `List`를 쓰면 안전할까, 아니면 별도의 설정이 필요한가?
    

### 핵심 연결

- Bean 다중 주입
    
- equals / hashCode
    
- 컬렉션 선택에 따른 Spring DI 결과

## 3. Collections 유틸리티 메서드와 불변 컬렉션

Spring 설정과 런타임 예외의 단골 지뢰입니다.

### 지뢰 질문

- `Collections.unmodifiableList()`로 감싼 컬렉션에 Spring이 값을 추가하려고 하면 언제, 어떤 예외가 발생하는가?
    
- 이 문제는 컴파일 타임에 잡히지 않는 이유는 무엇인가?
    
- `Collections.emptyList()`를 반환하는 메서드를 사용하는 설계는 어떤 경우에 안전하고, 어떤 경우에 위험해지는가?
    
- Spring 내부에서 불변 컬렉션을 사용하는 목적은 무엇이며, 실무 코드에서 이를 그대로 따라 써도 괜찮은가?
    

### 핵심 연결

- 불변 컬렉션
    
- UnsupportedOperationException
    
- 런타임 안정성
    
- Spring 내부 설계 의도

## 4. AbstractCollection / AbstractList의 존재 이유

직접 상속하지 않더라도 **설계 이해용 지뢰**로 가치가 있습니다.

### 지뢰 질문

- `AbstractCollection`은 왜 모든 메서드를 구현하지 않고 일부만 제공할까?
    
- `AbstractList`를 상속하면 어떤 메서드는 구현하지 않아도 동작하는 이유는 무엇인가?
    
- Spring 내부에서 커스텀 컬렉션을 직접 구현하지 않고 표준 구현체를 사용하는 이유는 무엇일까?
    
- 인터페이스를 바로 구현하는 것보다 Abstract 클래스가 제공하는 안정성은 무엇인가?
    

### 핵심 연결

- 템플릿 메서드 패턴
    
- 컬렉션 설계 철학
    
- 인터페이스 vs 추상 클래스

## 제외한 개념 (의도적으로 지뢰 질문 미생성)

- Collection 인터페이스의 메서드 목록  
    이유: Spring 및 실무에서 “암기 여부”보다 “선택과 계약 이해”가 중요
    
- List / Set / Map 기본 정의  
    이유: 이미 1회독 단계에서 충분히 소화 가능한 영역
