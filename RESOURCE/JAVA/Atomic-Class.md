# Atomic Class

Atomic 클래스들은 CAS 연산을 이용하여 락 없이 thread-safe 연산을 제공하는 동시성 유틸리티 클래스이다.

- CAS는 CPU가 제공하는 원자 연산이다
- Atomic 클래스는 [CAS](../CS/CAS.md) 기반이다
- CAS는 락을 쓰지 않는다
- 실패하면 다시 시도한다
- 락보다 빠를 수 있지만 항상 그런 건 아니다
- ABA 문제가 존재한다
- CAS는 만능이 아니다

## 대표 클래스

- `AtomicInteger`
- `AtomicLong`
- `AtomicBoolean`
- `AtomicReference`

### AtomicInteger

```java
AtomicInteger count = new AtomicInteger(0);  
int value = count.incrementAndGet(); // Integer의 count++; 와 동일한 연산
```

- synchronized나 Lock을 사용하지 않음
- CAS를 반복 시도


## 한계점

### ABA 문제

특정 위치를 두번 읽을 때, 두 번 읽은 값이 동일하면 첫 번째 스레드는 그 사이에 아무 일도 일어나지 않는다고 판단한다.

> 중간 변경을 파악하지 못하는 문제

#### 예시 

```
Thread 1 : 값이 A라면 B 로 변경
Thread 2 : 값을 B로 변경했다가 A로 변경
```

- Thread 1이 값을 읽음
- 그 사이에 Thread 2가 값을 B로 변경
- 다시 Thread 2가 값을 A로 변경
- Thread 1 입장에서는 값이 똑같이 A임 (중간에 변경이 있었음을 모름)
- Thread 1은 CAS를 성공시킴

#### 해결

```java
AtomicStampedReference<Integer> ref;
```

`AtomicStampedReference`를 사용하여 해결

## 적절한 사용 시점

- 단순 카운터
- 통계 값
- 상태 플래그
- 경쟁이 많지 않은 경우