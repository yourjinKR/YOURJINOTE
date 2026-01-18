# ReentrantLock

`ReentrantLock`은  `java.util.concurrent.locks` 패키지에서 제공하는 명시적(explicit) 락 구현체로, `synchronized`보다 유연한 락 획득·해제 및 조건 대기 메커니즘을 제공한다.

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

- true: 먼저 온 스레드 우선
- false: 성능 우선 (기본값)

### 조건 분리 (Condition)

```java
Condition condition = lock.newCondition();
```