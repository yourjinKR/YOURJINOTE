# 임계 영역 (Critical Section)

**임계영역(Critical Section)** 은 여러 스레드(또는 프로세스)가 **같은 공유 자원** 에 접근/수정하는 코드 구간 중에서, 동시에 실행되면 문제가 생길 수 있어 **한 번에 하나만 들어가야 하는 구간**을 말합니다.

## 임계 영역이 생기는 이유

### 공유 자원

멀티스레드 환경에서 여러 스레드가 하나의 공유 자원에 접근 할 수 있다.  
이때 자원 접근 순서가 보장되지 않으면 **경쟁 상태**가 발생한다.

### 원자성

아래와 같은 연산은 여러 로직을 거쳐 기존 값에 1을 증가시키는 증감연산자이다.  
특정 수식이 [원자적](원자성.md)이지 않을 경우에 발생한다.

```java
value++;
```

## 임계영역 발생 조건

### 실무 접근

- **공유 데이터/자원**가 있고
- **둘 이상의 스레드가 동시에 접근**하며
- 그 접근이 **쓰기(변경)** 를 포함하거나, 읽기라도 **일관성이 중요**하고
- 해당 동작이 **[원자적](원자성.md)(atomic)이지 않거나**, **가시성(visibility) 보장**이 없을 때

### 운영체제 이론

- Mutual Exclusion
- Progress
- Bounded Waiting

## 임계 영역에서 발생하는 문제

임계 영역에서 발생할 수 있는 문제들은 다음과 같다.
### Race Condition / Thread Interference

**경쟁 상태**

시스템의 실질적인 동작이 제어 할 수 없는 이벤트의 순서나 타이밍에 따라 달라지는 상태를 말한다.

> Thread Interference 라고도 부른다.

### Memory Visibility / Memory Consistency Errors

**메모리 가시성**

서로 다른 스레드가 동일해야 할 데이터에 대해 일관되지 않은 관점을 가질 때 발생합니다. 메모리 일관성 오류의 원인은 복잡하다.

> 원인에 대한 자세한 이유보다는 오류를 방지하는 전략을 아는 것이 중요하다.


이러한 문제들은 동기화를 통해 해결할 수 있으며, Java에서의 동기화는 다양한 방식으로 이뤄진다.

- [volatile](../JAVA/Java-Thread.md#volatile)
- [syncronized](../JAVA/Java-syncronized.md)
- [ReentrantLock](../JAVA/ReentrantLock.md)
- [Atomic-Class](../JAVA/Atomic-Class.md)