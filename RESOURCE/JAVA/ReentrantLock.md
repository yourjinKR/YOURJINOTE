# ReentrantLock

`ReentrantLock`은  `java.util.concurrent.locks` 패키지에서 제공하는 명시적(explicit) 락 구현체로, [synchronized](Java-syncronized.md)보다 유연한 락 획득·해제 및 조건 대기 메커니즘을 제공한다.

> `ReentrantLock`은 수동 조작이 가능한 스마트 락이다. `synchronized` 보다는 복잡하지만 유연하다는 강점을 지니고 있지만 완전한 대체제는 아니다.

```java
static class BankAccountReentrant implements BankAccountImpl {  
    private int balance = 0;  
    private final Lock lock = new ReentrantLock();  
  
    public void deposit(int amount) {  
        lock.lock();  
  
        try {  
            balance += amount;  
        } finally {  
            lock.unlock();  
        }  
    }  
  
    public int getBalance() {  
        return balance;  
    }  
}
```

- `new ReentrantLock()` 하여 Lock 객체 생성
- 반드시 `finally` 에서 `unlock()` 수행
- 개발자가 생명주기를 직접 관리

## 지원 핵심 기능

### 타임아웃

```java
// 1초 대기 후 포기
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        doWork();
    } finally {
        lock.unlock();
    }
}
```

### 인터럽트 가능한 락

```java
lock.lockInterruptibly();
```

- 대기 중 interrupt 시 즉시 반응

### 공정성(Fairness) 옵션

```java
new ReentrantLock(true);
```

- true : 먼저 온 스레드 우선
- false : 성능 우선 (기본값)

### Condition

Lock과 함께 사용되는 대기/신호 메커니즘으로, `Object`의 `wait`/`notify`를 대체하며 여러 개의 조건 큐를 명시적으로 관리할 수 있게 해준다.

[생산자-소비자 문제](../CS/생산자-소비자-문제.md)의 정석적인 해결 방법이다.

> 직접 lock/unlock을 제어하기에 실무에서 직접 구현하는 일은 드물다.  
> 실무에서는 [BlockingQueue](BlockingQueue.md)를 주로 활용

`Condition`은 기본적으로 `newCondition()` 메서드를 조건별로 호출하여 사용한다.

```java
Lock lock = new ReentrantLock();
// 조건별 생성
Condition notEmpty = lock.newCondition();
Condition notFull  = lock.newCondition();
```

```java
lock.lock();
try {
    while (buffer.isEmpty()) {
        notEmpty.await(); // 비어 있으면 소비자 대기
    }

    consume();

    notFull.signal(); // 생산자 깨우기
} finally {
    lock.unlock();
}
```

#### wait/notify vs Condition 비교

| 항목       | wait / notify | Condition |
| -------- | ------------- | --------- |
| 조건 큐     | 1개            | 여러 개      |
| 깨울 대상 선택 | ❌ 랜덤          | ⭕ 정확      |
| 가독성      | 낮음            | 높음        |
| 스푸리어스 대응 | 직접            | 구조적으로 안전  |
| 실무 사용    | ❌ 거의 안 씀      | ⭕ 적극 사용   |

