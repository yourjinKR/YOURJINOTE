# Executor

![Pasted image 20260126154025](../../GALLERY/Pasted%20image%2020260126154025.png)

Executor는 [스레드](RESOURCE/JAVA/Java-Thread.md) 생명주기와 작업 실행을 분리하여 서버 자원을 통제 가능한 구조로 만든 자바 동시성 프레임워크의 핵심이다.

# ExecutorService 

ExecutorService는 작업 제출과 실행을 분리하고, 작업 스케줄링, 실행, 종료를 관리하는 고수준 동시성 API이다.

### 생성

```java
ExecutorService executorService = Executors.newFixedThreadPool(4);
```

- 스레드 4개 고정
- 내부적으로 ThreadPoolExecutor 사용
- `Executors`의 팩토리 메서드로 간단히 생성 가능

### 작업 제출

```java
// Future<?> submit(Runnable task);  
executorService.submit(() -> {  
    System.out.println("작업 실행");  
});
```

- 스레드를 생성하지 않음
- 작업만 던진다

#### Runnable vs Callable

```java
Runnable r = () -> {};
```

- 리턴값이 없고 예외 처리에 제한

```java
Callable<Integer> c = () -> 1;
```

- 리턴값이 존재하며 예외를 던질 수 있음

### 종료

```java
// 미실행시 JVM 종료 불가  
executorService.shutdown();
```


##  Future

```java
Future<Integer> future = executorService.submit(() -> 10);  
```

- 비동기 실행
- 필요할 때 결과를 수령할 수 있음

## 과정 요약

![Pasted image 20260126155318](../../GALLERY/Pasted%20image%2020260126155318.png)

![Pasted image 20260126155322](../../GALLERY/Pasted%20image%2020260126155322.png)

![Pasted image 20260126155327](../../GALLERY/Pasted%20image%2020260126155327.png)

<br>

# 출처

[baeldung - java thread pool](https://www.baeldung.com/thread-pool-java-and-guava)
