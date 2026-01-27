# Lock

[임계 영역](Critical-Section-임계-영역.md)을 하나의 스레드에서만 접근할 수 있도록 보호하는 방식

## 락 범위에 따른 구분

락의 범위에 따라 [coarse-grained lock](coarse-grained%20lock.md)과 [fine-grained lock](fine-grained%20lock.md)으로 나뉜다.  
일반적으로 성능 향상을 위한 fine-grained Lock을 사용하지만, 락 분할로 인한 복잡성 증가와 데드락 가능성, 유지보수 비용 등을 함께 고려해야 한다. 

## Database

### Optimistic-Locking

> 낙관적 락, Application Lock이라고도 부른다

데이터베이스에 대한 변경이 드물게 발생하고, 충돌 가능성이 낮을거라는 **낙관적인 가정** 하에 실제로 충돌이 발생했을 때만 대응하는 방식이다.

- 데이터 읽기에 락을 걸지 않음
- 이전 데이터와 현재 데이터를 비교하여 충돌 여부를 판단
- 데이터베이스의 **성능 저하를 최소화**, **동시성**을 높이는데 유리
- 애플리케이션에서 **사용자의 요청을 처리**할때 주로 사용

#### 대표 구현 방식

- 버전 번호 사용 (JPA의 `@Version`)

### Pessimistic-Locking

> 비관적 락, Database Transaction Lock 이라고도 부른다

데이터에 대한변경이 자주 발생하고 충돌 가능성이 높다는 **비관적 가정**을 기반으로 데이터를 처음부터 잠그는 방식으로 충돌을 방지한다.

- 데이터 읽을 때 부터 락을 건다
- **일관성 유지**에 초점
- **성능저하**와 **데드락 발생 가능성** 높음
- 금융 시스템과 같은 곳에서 주로 사용

#### 대표 구현 방식

- 데이터베이스 (SQL의 `SELECT FOR UPDATE`)


# 출처

[10분 테코톡](https://www.youtube.com/watch?v=LDi5muN2kgI)  
[f-lab](https://f-lab.kr/insight/understanding-optimistic-and-pessimistic-locking)  