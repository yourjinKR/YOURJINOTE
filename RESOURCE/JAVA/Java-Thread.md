# 자바의 Thread

Java에서 스레드는 **멀티 스레드**를 지원하여 하나의 프로세스 안에 한 개 이상의 스레드를 지원하는 구조이다. 이러한 멀티 스레드를 통해 **비동기식 및 병렬 애플리케이션**을 개발할 수 있다. 

![Pasted image 20260115202410](../../GALLERY/Pasted%20image%2020260115202410.png)

스레드를 구현하는 방법은 `Thread` 클래스를 상속 받거나 `Runnable` 인터페이스를 구현하는 방식으로 나뉜다. 객체지향 프로그래밍 측면에서 보통 `Runnable`로 구현하는 것을 권장한다.

## Thread

```java
public class MyThread extends Thread {  
    @Override  
    public void run() {  
        System.out.println("hello");  
    }  
}
```

- `run()`을 오버라이드하여 사용

```java
Thread thread = new MyThread();  
thread.start();
```

## Runnable

```java
public class MyRunnable implements Runnable {  
    @Override  
    public void run() {  
        System.out.println("hello");  
    }  
}
```

- `run()` 메서드만 구현하여 사용 가능
- `Runnable`은 스레드가 아닌 실행 가능한 작업이다.

```java
MyRunnable runnable = new MyRunnable();  
Thread thread = new Thread(runnable);  
thread.start();
```

## 스레드 실행

### run()

```java
thread.run();
```

- 일반 메서드 호출
- 새로운 스레드를 생성하는 개념이 아님
- 호출한 스레드가 실행

> `run()`은 스레드를 실행하는 것이 아닌 단순히 클래스에 선언된 메서드를 실행하는 것이다.

### start()

```java
thread.start();
```

- JVM에게 아래와 같은 요청
	- 새 스레드를 생성 후 해당 객체의 `run()`을 실행
- 실제 스레드를 생성
- 호출 즉시 반환

> `start`를 호출한다고 해서 바로 스레드가 실행되는 것이 아닌, 실행 대기 상태에 있다가 실행할 순서에 맞게 실행된다. 스레드의 실행 순서는 OS의 스케줄러가 관리한다.

```java
Thread thread2 = new MyThread();  
thread2.start();  
thread2.start(); // IllegalThreadStateException
```

한번 실행이 종료된 스레드는 다시 실행시킬 수 없으며 아래와 같이 호출하면 예외가 발생한다.

#### 내부 로직

- main 메서드에서 스레드의 `start()`를 호출
- `start()`는 새로운 스레드를 생성하고 스레드가 작업하는데 사용될 호출스택을 생성
- 새로 생성된 호출스택에 `run()`이 호출되어 스레드가 독립된 공간에서 작업을 수행
- 여러 개의 호출스택은 스케줄러가 정해준 순서에 의해 번갈아가며 실행

### main Thread

메인 메서드의 작업을 수행하는 스레드

### start() vs run()

```
main thread
   |
   |-- thread.run()  -> 같은 실행 흐름
   |
   |-- thread.start()
          |
          +-- new thread 생성
                  |
                  +-- run() 실행

```

### Thread.currentThread()

> 현재 실행중인 스레드의 참조를 반환하는 메소드

```java
public final String getName() {  
    return name;  
}
```

`Thread` 클래스에는 현재 스레드의 이름을 불러오는 `getName()` 메서드가 존재한다. 

```java
public class Thread implements Runnable {
	@IntrinsicCandidate  
	public static native Thread currentThread();
}
```

```java
public class MyRunnable implements Runnable {  
    @Override  
    public void run() {  
        System.out.println(Thread.currentThread().getName());  
    }  
}
```

`Runnalbe`에는 `run()` 추상   메서드만이 존재하기에 `Thread` 클래스에서 기본적으로 제공하는 메소드를 사용하기 위해서는 `currentThread()` 정적 메서드를 통해 현재 스레드의 참조를 불러와야 한다.

## 스레드의 우선순위

`Thread` 내에는 `priority`라는 멤버 변수로 우선순위를 표현한다.  
`main` 메서드 내에서 생성된 `Thread`들은 자동적으로 `5`이다.

> 우선순위를 차등적으로 설정하여 중요한 작업의 스레드가 더 많은 시간을 점유한다.

```java
public static final int MIN_PRIORITY = 1;
public static final int NORM_PRIORITY = 5;
public static final int MAX_PRIORITY = 10;
```

## 스레드 그룹

서로 비슷한 성질을 지닌 스레드들을 관리하기 위한 것이다.  
자신이 속한 스레드 그룹이거나 하위 그룹은 변경할 수 있으나 그 외에는 변경하지 못한다. 

```java
public ThreadGroup(String name);
public ThreadGroup(ThreadGroup parent, String name);
// 등등...
```

모든 스레드는 스레드 그룹에 포함되어야 한다. 그러므로 스레드 그룹을 지정하는 생성자를 사용하지 않은 스레드는 기본적으로 자신을 생성한 스레드와 같은 스레드 그룹에 속하게 된다.

## 생명 주기와 상태 전이

![Pasted image 20260115202445](../../GALLERY/Pasted%20image%2020260115202445.png)

자바 스레드는 항상 6가지 상태 중 하나에 있으며 여러 상태를 거치며 생명 주기를 보낸다.
- new
- runnable
- blocked
- waiting
- time-waiting
- terminated

```java
public enum State {
	NEW,
	RUNNABLE,
	BLOCKED,
	WAITING,
	TIMED_WAITING,
	TERMINATED;
}
    
public State getState() {  
    // get current thread state  
    return jdk.internal.misc.VM.toThreadState(threadStatus);  
}
```

### NEW

```java
Thread t = new Thread(task);
```

- 객체 생성
- 아직 스레드가 아님

### RUNNABLE

```java
t.start();
```

- 실행 가능 상태
- 실행 중이거나 대기 중인 상태

### BLOCKED

```java
synchronized (lock) {
    // 임계 영역
}
```

- `synchronized` 락을 기다릴 때 BLOCKED 상태가 된다
- 이미 다른 스레드가 lock을 잡고 있음
- 들어가려다 **BLOCKED**
- BLOCKED는 **락 전용 상태**
- 개발자가 직접 제어 불가

### WAITING / TIMED_WAITING

```java
Thread.sleep(1000);
```

```java
join(ms);
```

- 시간 조건이 있는 대기
- `WAITING` : 무기한
- `TIMED_WAITING` : 시간 만료 시 자동 복귀

### TERMINATED

- `run()` 종료시
- 예외로 종료시
- 정상 종료시

## 스레드 실행 제어
### sleep

```java
Thread.sleep(1000);
```

- 현재 스레드를 일시 정지
- 락을 반납하지 않으며 다른 스레드가 BLOCKED
- `synchronized` 안에서 사용시 위험

### join

```java
thread.join();
```

- 해당 스레드가 끝날 때까지 `main`이 대기
- `join`은 상대 스레드를 멈추는 게 아닌 **현재 스레드**가 WAITING

### Join과 Sleep의 필요성

```java
Thread t = new Thread(() -> {  
    System.out.println("스레드 실행");  
});  
  
t.start();  
System.out.println("메인 종료");

/*
메인 종료
스레드 실행
*/
```

스레드는 알아서 순서를 맞춰주지 않는다.  
위와 같은 간단한 코드 또한 순서를 예측하기 어렵다.

#### 잘못된 대기 방식

```java
while (t.isAlive()) {
    // 기다린다
}
```

위와 같이 조건문을 통해 실행에 순서를 정할 수 있지만 해당 방식에는 많은 문제점이 있다.

- CPU를 계속 사용
- 서버 성능 저하

### interrupt

해당 스레드의 작업을 멈추라고 요청하는 메서드

```java
public void interrupt() // interrupted 상태를 true로 변경
public boolean isInterrupted() // interrupted를 반환
public static boolean interrupted() // interrupted 상태를 반환하고 true로 변경
```

스레드가 sleep(), wait(), join()으로 인해 WAITING 상태일때 해당 스레드에 대해 `interrupted()` 호출시 `InterruptedException`이 발생하고 스레드는 RUNNABLE 상태로 바뀐다.

#### InterruptedException 주의사항

```java
Thread t = new Thread(() -> {  
    try {  
        Thread.sleep(10000);  
    } catch (InterruptedException e) {  
        System.out.println("인터럽트 감지");  
    }  
});  
  
t.start();  
Thread.sleep(1000);  
t.interrupt();
```

`InterruptedException`가 발생하면 `interrupted`가 초기화된다. (`interrupted = false;`)  
즉, 종료 요청이 무시되고 스레드가 계속 실행된다.  

#### 강제 종료 기능이 없는 이유

자바에서는 스레드의 강제 종료 기능을 지원하지 않는다.  
강제 종료의 문제점은 다음과 같다.  

- 락을 쥔 채로 종료
- 데이터 중간 상태
- `finally` 미실행
- 리소스 누수

자바에서 스레드의 종료 방식은 `interrupt`를 통해 요청을 보내고 실질적인 종료는 스레드가 책임지도록 설게했다. 

이를 **협력적 중단**이라고 한다.

```java
Thread t = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        // 작업 수행
    }

    // 정리 작업
    System.out.println("정상 종료");
});
```


# 동시성

자바에서 동시성은 여러 스레드가 동시에 실행될 수 있도록 하여 성능과 효율성을 향상시킨다.

## 동시성 이슈

그러나 공유 데이터를 부적절하게 다루면 프로그램 동작에 심각한 문제가 발생할 수 있다.

## Race Condition (경쟁 상태)

스레드는 실행되고 있는 CPU 메모리 영역 내 데이터를 캐싱한다.  

그로 인해 아래와 같은 문제가 발생한다.  
- 캐싱된 시점에 따라 다수의 스레드가 특정 변수를 공유하더라도 값이 다를 수 있다.
- 캐싱된 데이터가 언제 갱신되는지 정확히 알 수 없다.

### volatile

> volatile : 휘발성 물질

Volatile 키워드는 자바에서 변수의 가시성을 보장하기 위해 사용된다.  
이는 특정 변수의 값을 **메인 메모리에 직접 저장하고 읽도록 강제**한다.

```java
static volatile boolean stop = false;
```

#### 문제점

```
1) 변수 a =1
2) 1번 스레드가 a에 값을 증가시키기 위해 a의 값을 읽습니다. a=1
3) 1번 스레드가 1 + 1을 계산하여 2를 얻습니다. (아직 저장은 안 함)
4) 2번 스레드가 a에 값을 증가시키기 위해 a의 값을 읽습니다. a=1 
5) 1번 스레드가 변수 a에 2를 저장
6) 2번 스레드가 1 + 1을 계산하여 2를 얻습니다. (아직 저장은 안 함)
7) 2번 스레드가 변수 a에 2를 저장
```

### synchronized

`synchronized` 키워드는 자바에서 임계 영역을 보호하기 위해 사용된다.  
이는 특정 코드 블록이나 메서드에 대해 **하나의 스레드만 접근할 수 있도록 보장**한다.

`synchronized`는 가시성과 [원자성](원자성.md)을 모두 보장한다.  

```java
public class SynchronizedEx {  
    private int count = 0;  
  
    public synchronized void increment() {  
        count++;  
    }  
  
    public synchronized int getCount() {  
        return count;  
    }  
}
```

> `synchronized`는 한 번에 하나의 스레드만 들어오게 만드는 것이다.  
> 그로 인해 해당 방식은 성능 저하로 이어질 수 있다.

자세한 내용은 [여기](Java-syncronized.md)를 참고

### Volatile과 Synchronized의 차이점

Volatile과 Synchronized는 모두 멀티스레드 환경에서 데이터의 일관성을 유지하기 위한 도구이다.

`volatile`은 변수의 가시성을 보장하지만, 원자성을 보장하지 않는다.
반면, `synchronized`는 가시성과 원자성을 모두 보장한다.  

읽기 작업이 많은 경우는 `volatile`, 읽기와 쓰기 작업이 혼합된 경우는 `syncronized`가 적합하다.

왜냐하면 Synchronized는 락을 사용하여 임계 영역을 보호하기 때문에, 여러 스레드가 동시에 동일한 자원에 접근하는 것을 방지할 수 있기 때문이다.

> 과도한 락의 사용은 성능 저하와 [데드락](../CS/deadlock.md)과 같은 문제가 발생한다.

## Dead Lock (교착 상태)

[교착상태](../CS/deadlock.md)란 두 개 이상의 스레드가 서로 리소스 해제를 기다리면서 모든 스레드가 영원히 멈춰버리는 현상이다.

```java
// Thread-unsafe  
private volatile boolean running = true;  
  
void start() {  
    new Thread(() -> {  
        while (running) {}  
        System.out.println("Stopped!");  
    }).start();  
}  
  
// Change may not be visible to other thread  
void stop() {  
    running = false;  
}  
  
public static void main(String[] args) throws InterruptedException {  
    DeadLock2 t = new DeadLock2();  
    t.start();  
  
    // Short pause before stopping  
    Thread.sleep(100);  
  
    // Thread may not see this without volatile  
    t.stop();  
}
```

## Starvation (기아)

- 특정 프로세스의 우선순위가 낮아 원하는 자원을 영원히 할당받지 못하는 상태
- 해당 현상은 스레드의 동기화 이슈에서만 발생하는 문제점은 아님

### 해결방안

fair lock 또는 적절한 스레드 스케줄링 매커니즘을 사용
<br>

# 출처

[flab - volatile와 synchronized](https://f-lab.kr/insight/volatile-and-synchronized-in-java-20250620)  
[geeksforgeeks - Thread](https://www.geeksforgeeks.org/java/java-threads/)  
[geeksforgeeks - concurrency problem](https://www.geeksforgeeks.org/java/concurrency-problems-in-java/)