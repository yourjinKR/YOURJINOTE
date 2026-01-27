# Concurrent Collection

`Concurrent` 컬렉션은  `java.util.concurrent` 패키지에서 제공하는 [[RESOURCE/JAVA/Java-Thread.md|thread]]-safe 컬렉션 구현체로, 내부적으로 세분화된 락([fine-grained lock](../CS/fine-grained%20lock.md)) 또는 락-프리 기법을 사용하여 높은 동시성과 성능을 제공한다.

- `CopyOnWriteArrayList` : 읽기 많고 쓰기가 적을 때 최적인 컬렉션

- `ConcurrentHashMap`

- `ConcurrentSkipListMap`
    
- `ConcurrentLinkedQueue`
    
- `ConcurrentSkipListSet`

## ConcurrentHashMap

`ConcurrentHashMap`은 해시 테이블 기반의 thread-safe Map 구현체로, 버킷 단위 또는 내부 노드 단위로 락을 분리하거나 [CAS](../CS/CAS.md) 기반 알고리즘을 사용하여 높은 동시성을 제공한다.

> **전체 락**이 아닌 **부분 락**을 사용한다

```java
Map<String, Integer> map = new ConcurrentHashMap<>();
map.put("a", 1);   // thread-safe + 고성능
```

### Java 7 구조

- Map을 여러 Segment로 분할
- Segment마다 락 하나

> 동시에 여러 스레드 접근 가능하다

### Java 8+ 구조

- Segment 제거
- 버킷(bin) + 노드 단위 락
- 대부분 연산:
    - 읽기 → 락 없음
    - 쓰기 → 필요한 부분만 락

### 주요 특징

#### 1. 읽기는 락이 거의 존재 하지 않음

- `get()`은 대부분 락 프리
- 읽기 스레드끼리는 충돌 거의 없음

#### 2. put / remove는 부분 락

- 해당 버킷만 잠금
- 다른 버킷은 동시 접근 가능

#### 3. null 비허용

```java
Map<String, Integer> map = new ConcurrentHashMap<>();  
map.put("null", null); // NullPointerException
```

- 값에 `null`을 삽입할 경우 NPE 발생
- `null`이면  값이 없는건지 `null` 자체가 값인지 구분하기 힘들기 때문에

#### 4. size() 불일치 가능성

- Java 8 이후로는 대부분 개선
- `size()`, `isEmpty()`는 그 **순간 값**일 뿐 정확한 동기화 값이 아님


## 기존 방식의 문제점

### 일반적인 컬렉션 프레임워크

```java
Map<String, String> map = new HashMap<>();
```

- 내부 버킷 구조 깨짐
- 무한 루프 발생 가능
- CPU 버그 (ex : HashMap resize 버그)

`HashMap`은 멀티스레드에서 절대 사용을 금지한다

### Collections.synchronizedMap

```java
Map<String, String> map =
    Collections.synchronizedMap(new HashMap<>());
    
synchronized (map) {
    for (String key : map.keySet()) {
        ...
    }
}
```

- 모든 메서드에 `synchronized` 자동 적용
- thread-safe

- 단일 락 구조로 전체가 잠김 → 병렬성 저하, 성능 저하 유발
- 읽기 / 쓰기 락 구분 없음 → Read-heavy 환경에서 비효율적
- 반복(iteration) 시 외부 `synchronized` 필요 → 코드 복잡성 및 버그 위험 증가