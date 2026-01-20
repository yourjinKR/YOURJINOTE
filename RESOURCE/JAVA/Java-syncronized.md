# syncronized

`syncronized`는 **객체 모니터**를 이용하여 한 시점에 하나의 [스레드](Java-Thread.md)만 **임계 영역**에 진입하도록 보장하는 자바의 내장 동기화 매커니즘이다.

마치 공용 물건을 쓰기 위해 '열쇠'를 하나만 두는 규칙과 같은 개념이다.

## 주요 키워드

- **객체 모니터 (monitor)**
- **임계 영역 (critical section)**
- **상호 배제 (mutual exclusion)**
- **메모리 가시성 (visibility, happens-before)**

### 객체 모니터 

- 모든 자바 객체는 모니터를 하나씩 가진다
- `synchronized`는 그 모니터를 획득해야 사용 가능

### 임계 영역

여러 스레드가 동시에 실행하면 데이터가 깨질 수 있는 코드 영역

```java
count++; // 대표적인 임계 영역
```

> `++` 연산자는 여러 동작을 통해 작동한다.  
> `count` 값을 읽고, 수정하고, 덮어쓰는 과정을 가진다.

### 상호 배제

한 시점에 하나의 스레드만 실행

- 다른 스레드는 BLOCKED
- 순서를 보장하지 않음
- 동시 실행만 차단함

### 메모리 가시성

`synchronized` 블록의 진입/종료는 happens-before 관계를 형성한다.

- 진입시 : 최신 값을 읽는다
- 종료시 : 변경 사항 메모리에 반영


## synchronized 메서드 vs 블록

실무에서는 블록 방식을 선호
- 필요한 부분만 보호
- 락 범위 최소화 → 성능 향상

### 메서드

```java
public class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }
}
```

- `this` 객체의 **모니터** 획득
- increment() 전체가 **임계 영역**
- 한 번에 하나의 스레드만 진입
- count 변경은 **원자성 + 가시성 보장**

### 블록

```java
public class Counter {
    private int count = 0;

	public void increment() {
	    synchronized (this) {
	        count++;
	    }
	}
}
```

- 원하는 객체에 LOCK (현재 코드는 `this`)
- `synchronized` 블럭 내부가 **임계 영역**
- 한번에 하나의 스레드만 진입
- count 변경은 **원자성 + 가시성 보장**

## 한계점

```java
synchronized (lock) {
    doWork();
}
```

`synchronized`는 개발자가 락의 상태를 직접 제어할 수 없다.  

아래는 `synchronized` 방식의 대표적인 한계점 4가지이다.

- 타임아웃, 중간취소와 같은 중도 포기가 불가능
- interrupt에 반응하지 않음
- 조건별 대기 불가능
- 락 상태를 확인 할 수 없다.

이를 해결하기 위해 [ReentrantLock](ReentrantLock.md)를 사용한다.


<br>

# 출처

[geeksforgeeks](https://www.geeksforgeeks.org/java/concurrency-problems-in-java/)  