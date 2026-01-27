# ThreadPool

ThreadPool은 **미리 생성된 스레드 집합을 이용**하여 작업(Runnable / Callable)을 **재사용 가능한 스레드**에 할당해 처리하는 실행 프레임워크이다.

## 사용 목적

> `new Thread()`를 통해 스레드를 직접 관리하는 것은 마치 식당 사장이 주문마다 알바생을 고용하는 행위와 비슷하다

스레드를 사용하려면 먼저 [[RESOURCE/JAVA/Java-Thread.md|스레드]]를 생성해야 하지만 다음과 같은 문제가 존재

- 스레드 생성 시간으로 인한 성능 저하 문제
- 스레드 관리 문제
- Runnable 인터페이스의 불편성

자세한 내용은 [해당 글](https://hyuuny.tistory.com/245)을 참고


## 4대 구성 요소

```
[작업 큐] → [스레드 풀] → [실행] → [결과]
```

1. 작업 큐
2. 스레드 집합
3. 작업
4. 실행 관리자 ([Executor](RESOURCE/JAVA/Executor.md))

## 종류

### fixedThreadPool

```java
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(4);  
```

- 스레드 수 고정
- **가장 많이 사용**

### cachedThreadPool

```java
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();  
```

- 스레드 무제한 증가 가능 (서버 환경에서는 부적합)

### singleThreadExecutor

```java
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();  
```

- 항상 하나의 스레드
- **순서 보장**

### scheduledExecutorService

```java  
ExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(2);
```

- 주기 작업에 주로 사용

## ThreadPoolExecutor

실무에서는 Executors 팩토리를 사용하지 않고 직접 `ThreadPoolExecutor`를 생성하여 사용한다.

> Executors 팩토리는 큐가 무제한이고 OOM(Out Of Memory) 위험이 존재함

```java
ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(  
        4, 8,  
        60, TimeUnit.SECONDS,  
        new ArrayBlockingQueue<>(100)  
);
```

![Pasted image 20260126151647](GALLERY/Pasted%20image%2020260126151647.png)


직접 객체를 생성하여 사용한다면 아래 상태들을 직접 관리 가능하다.

- 스레드 수
- 큐 크기
- 거절 정책 설정

<br>

# 출처

[baeldung - java thread pool](https://www.baeldung.com/thread-pool-java-and-guava)