# Runtime Data Area란?
- [JVM](JVM.md)이 읽어 들인 각종 타입 정보, 상수, 정적 변수 정보가 저장되는 영역
- [Java-JIT-컴파일러](Java-JIT-컴파일러.md)가 번역한 **기계어 코드를 캐싱**하기 위한 메모리 공간으로 활용
- Java 8부터는 PermGen이 아니라 Metaspace에 속한다.

# 영역
![Pasted image 20251203195758](../../GALLERY/Pasted%20image%2020251203195758.png)

- 메소드 영역(Method Area)
- 힙(Heap)
- 스택(Stack)
- PC 레지스터(Program Counter Register)
- 네이티브 메소드 스택(Native Method Stack)

# 메소드 영역 (Method Area)
- JVM이 읽어 들인 각종 타입 정보, 상수, 정적 변수 정보가 저장되는 영역
- JIT 컴파일러가 번역한 기계어 코드를 캐싱하기 위한 메모리 공간으로 활용
- Java 8부터는 PermGen이 아니라 **Metaspace**에 속함

> **Metaspace**
> 메타스페이스는 JVM 힙이 아니라 네이티브 메모리에서 관리하며 크기가 동적으로 달라질 수 있다.
## 상수 풀 (Rutime constant pool)
- 클래스 버전, 필드, 메서드, 인터페이스 등 클래스 파일에 포함된 정보 및 각종 리터럴, 심볼 참조가 저장되는 영역
- **클래스 로더가** 클래스를 로드할 때 상기 **정보들을 저장**
- **동적으로 운영**되며 런타임에 새로운 상수가 추가될 수 있음

> 해당 값들은 모든 쓰레드가 공유한다.

### 상수 풀 예시
![Pasted image 20251203200132](../../GALLERY/Pasted%20image%2020251203200132.png)

# 힙-Heap
- [가비지 컬렉터](Java-가비지%20컬렉션-Garbage-Collection.md)가 관리하는 메모리 영역으로 Java에서 사용되는 **객체**(인스턴스)가 저장되는 공간
- 설정에 따라 크기를 변경하거나 고정 가능
	- 부족 시 `OutOfMemoryError` 오류 발생
- 세대별 컬렉션 이론(Generational collection theory)을 기반으로 설계 및 운영

> `new` 키워드를 통해 동적으로 생성된 인스턴스 객체가 저장되는 영역으로 
> 모든 쓰레드가 공유하며, **Garbage Collection**의 대상이 되는 영역이다

> Heap 영역에 있는 인스턴스를 접근하기 위해서는 Stack에 저장되어 있는 Reference를 통해 접근

![Pasted image 20251203202355](../../GALLERY/Pasted%20image%2020251203202355.png)

## 세대 단위 컬렉션 이론
### 개요 : 세대 단위 컬렉션 이론 기초 가설
- 약한 세대 가설 : 대다수 객체는 일찍 죽는다.
- 강한 세대 가설 : 가비지 컬렉션 과정에서 살아남은 횟수가 늘어날수록 더 오래 살 가능성이 커진다.
- 세대 간 참조 가설 : 세대 간 참조의 개수는 같은 세대 안에서 참조보다 훨씬 적다.

Heap 영역은 효율적인 Garbage Collection을 위해 크게 3가지 영역으로 구분된다.

![Pasted image 20251204204444](../../GALLERY/Pasted%20image%2020251204204444.png)
### Young Generation
새롭게 생성된 객체가 할당되는 영역
대부분 객체가 금방 Unreachable한 상태가 되기 때문에, 많은 객체가 Young 영역에 생성되었다가 사라짐.
Young 영역에 대한 가비지 컬렉션을 Minor GC라고 부른다.
#### Minor GC
- new 키워드를 통해 새로운 인스턴스가 생성되면 Eden영역에 저장
- 이후 Servivor로 이동
- 시간이 지나면서 이 영역에 있는 데이터는 우선순위에 따라 Old 영역으로 이동 혹은 GC에 의해 수거
### Tenured(Old) Generation
Young Generation영역에 저장되었던 객체 중 오래된 인스턴스가 이동되어 저장되는 영역.
**Young 영역보다 크게 할당**되며, 영역의 크기가 큰 만큼 가비지는 적게 발생한다.
Old 영역에 대한 가비지 컬렉션을 Major GC 또는 Full GC라고 부른다.
#### Major GC
- Young 영역에서 살아남은 객체가 Old 영역에 복사
- Old 영역에 할당된 메모리가 허용치를 넘게 되면, Old 영역에 있는 모든 인스턴스들을 검사
- 참조되지 않는 인스턴스를 한꺼번에 삭제
- Major GC가 발생하면 [STW](STW-Stop-The-World.md) 발생

### Permanent Generation
ClassLoader에 의해 동적으로 로딩된 클래스의 메타데이터가 저장되는 영역
이 정보들은 JVM 실행 도중에 변경되지 않으며, JVM 종료 시까지 유지

> Java 8 이후 **Metaspace로 대체**되어 **Heap영역에서 제외**되었다

## Metaspace
- Perm 영역에서 저장하던 Class의 Meta 정보들이 이 영역에서 저장
- Native Memory 영역에 위치하며 JVM이 아닌 운영체제가 관리
- 클래스 메타데이터와 리플렉션을 이용하는 애플리케이션에서 사용하는 일부 메모리를 저장

### Perm 영역을 대체한 이유
- Perm 영역의 메모리 누수, OutOfMemoryError 등과 같은 문제
    - 클래스 로딩 및 언로딩 과정에서 메모리 할당 및 해제의 빈번한 발생으로 인해 이러한 문제가 더 심각해졌다.
- 클래스 메타데이터를 Native Memory에 저장하면서,  
    JVM에서의 OutOfMemoryError 문제가 해결되었다.

# 스택 (Stack)
- 지역변수 테이블, 피연산자 스택, 메서드 반환 값 등을 저장
- 보통 지역변수 테이블을 스택으로 지칭
- 지역 변수 테이블은 슬롯으로 이루어지며 기본형 변수 하나가 슬록  한 개를 사용
- Java 스택의 크기는 메모리 용량이 아니라 슬롯의 개수
- JVM이 허용하는 스택의 크기를 초과할 경우 `StackOverflowError` 에러 발생

> 슬롯의 용량이 32bit라고 가정, 8bit인 byte 데이터를 넣더라도 byte 변수 하나가 하나의 슬롯을 차지함

## Stack frame 구조
![Pasted image 20251203203939](../../GALLERY/Pasted%20image%2020251203203939.png)
### 상수 풀 참조 (Constant Pool Reference)
- 메소드가 속한 클래스의 상수를 사용하기 위해 Runtime Constant Pool에 대한 참조값을 가진다
### 지역 변수 테이블 (Local Variables Array)
- 지역변수 테이블은 인덱스 방식으로 운영
- 첫번째 인덱스에는 현재 인스턴스에 대한 참조값을 저장(this)
- 두번째 인덱스부터 매개변수 -> 지역변수 순으로 값을 저장
### 피연산자 스택 (Operand Stack)
JVM은 **Stack기반으로 연산**을 수행한다.

따라서 연산에 필요한 operand가 모두 Stack에 저장되어있기 때문에 Byte Code는 일반적인 어셈블리어와 달리 operand를 지정하지 않아도 연산이 가능하다. 

다른 어셈블리어와 달리 연산에 레지스터를 쓰지 않은 이유는 **각 디바이스마다 레지스터의 수가 달라** 그 수를 가정할 수 없기 때문에 **하드웨어의 관여를 최소화**하기 위해 JVM에서는 연산과정이 복잡하더라도 Stack을 사용한다

# PC Register

> Stack, Native Method Stack과 함께 쓰레드가 생성될 때 각 쓰레드마다 할당되는 영역으로, 
> 현재 수행중인 JVM의 명령어 주소를 저장한다

JVM은 Stack-Base방식으로 작동하기 때문에 Stack에서 Operand를 뽑아내에 별도의 메모리 공간에 저장한 다음 연산을 진행하는데, 이때 사용되는 메모리공간이 PC Register이다.

PC Register에는 멀티 쓰레드 프로그래밍 환경에서 한 쓰레드가 작업을 하다가 다른 쓰레드로 CPU자원을 넘겨주고 다시 받았을 때, 이어서 작업을 하기 위해 현재 실행중인 명령어의 주소를 기록한다.

만약 쓰레드가 JVM Instruction을 수행중이라면 Instruction의 주소를 가지고 있지만, Native Method를 수행중이라면 PC Register는 Undefined상태로 남아있고, 이에 대한 처리는 Native Method Stack에서 담당한다

# Native Method Stack

> Native 언어(C/C++, 어셈블리)로 작성된 코드를 실행하기 위한 메모리 영역

- 자바 이외의 언어가 JVM에서 동작하기 위해 할당한 메모리 영역으로, 일반적으로 C스택을 사용한다
- 쓰레드에서 java메소드가 아닌 Native방식을 사용하는 메소드를 실행하면 이 곳에 해당 메소드에 대한 정보를 저장한다
- JNI(Java Native Interface)를 통해 표준에 가까운 방식으로 구현이 가능하다.

# 출처
[인프런 강의](https://www.inflearn.com/course/%EB%8F%85%ED%95%98%EA%B2%8C-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-java-part2/dashboard)

https://velog.io/@impala/JAVA-JVM-Runtime-Data-Area

