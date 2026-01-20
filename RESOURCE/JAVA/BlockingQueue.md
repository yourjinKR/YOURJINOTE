# BlockingQueue

BlockingQueue는 큐가 비어 있거나 가득 찼을 때 자동으로 [스레드](Java-Thread.md)를 대기시키는 thread-safe 큐 인터페이스이다. 

- BlockingQueue는 [생산자–소비자](../CS/생산자-소비자-문제.md) 전용 도구다
- put / take는 **자동으로 block**된다
- 내부에서 **Lock + Condition을 다 처리**해준다
- Condition 직접 구현은 거의 금지
- **ThreadPool 내부**도 BlockingQueue를 쓴다
- 실무 표준은 `ArrayBlockingQueue` / `LinkedBlockingQueue`

## 구현체

### ArrayBlockingQueue

```java
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);
```

- 고정 크기
- 가장 기본
- 성능 안정적

### LinkedBlockingQueue

- 크기 가변 (기본 무한)
- 메모리 주의
- Executor 내부 기본 큐

### SynchronousQueue

- 저장 공간 0
- 넣자마자 바로 가져가야 함
- ThreadPool 고급 전략용

## 주요 메서드

### put/take - 완전 블로킹

```java
// 생산
queue.put(1);   // 가득 차면 자동 대기
// 소비
int value = queue.take();  // 비어 있으면 자동 대기
```

### offer / poll - 안 기다림

```java
boolean offer = queue.offer(2);  
Integer poll = queue.poll();
```

#### 타임아웃 버전

```java
boolean offer1 = queue.offer(3, 1, TimeUnit.SECONDS);  
Integer poll1 = queue.poll(1, TimeUnit.SECONDS);
```