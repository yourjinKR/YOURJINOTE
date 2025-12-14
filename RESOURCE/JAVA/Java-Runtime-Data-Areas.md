# Runtime Data Area란

[JVM](JVM.md)이 운영체제로부터 할당받은 메모리 영역

![Pasted image 20251203195758](../../GALLERY/Pasted%20image%2020251203195758.png)

- 메소드 영역(Method Area)
- 힙(Heap)
- 스택(Stack)
- PC 레지스터(Program Counter Register)
- 네이티브 메소드 스택(Native Method Stack)

## 요약

- 정적 변수는 메서드 영역에 저장
- 인스턴스 변수는 힙에 저장
- 지역 변수는 스택에 저장

# 메소드 영역 (Method Area)

- 힙의 논리적 부분이며 JVM이 시작될 때 생성
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

# Heap

- Java에서 사용되는 객체와 배열이 저장되는 공간이며 [가비지 컬렉터](Java-가비지-컬렉션-Garbage-Collection.md)가 메모리 관리
- 설정에 따라 크기를 변경하거나 고정 가능
	- 부족 시 `OutOfMemoryError` 오류 발생
- [세대 단위 컬렉션 이론](Java-세대-단위-컬렉션-이론-Generational-collection-theory.md)을 기반으로 설계 및 운영
- 핫스팟 VM에서 에덴과 생존자 공간 비율은 보통 8:1
	- 보통 첫 GC에 98%의 객체가 소멸된다.
	- 10%는 메모리 파편화 방지를 위해 활용된다.

> 모든 쓰레드가 공유하며, 실행 중인 JVM 프로세스에는 오직 하나의 힙만 존재한다.
> 
> `new` 키워드를 통해 동적으로 생성된 인스턴스 객체는 힙에 할당되고 해당 참조는 스택에 저장된다.  
> Heap 영역에 있는 인스턴스를 접근하기 위해서는 Stack에 저장되어 있는 Reference를 통해 접근한다.


![Pasted image 20251203202355](../../GALLERY/Pasted%20image%2020251203202355.png)

# 스택 (Stack)

- 지역변수 테이블, 피연산자 스택, 메서드 반환 값 등을 저장
- 보통 **지역변수 테이블**을 스택으로 지칭
- 지역 변수 테이블은 **슬롯**으로 이루어지며 기본형 변수 하나가 슬록  한 개를 사용
- 각 스레드가 자체 스택을 가지고 있기에 **스레드 안정성**을 보장한다.
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
[geeksforgeeks](https://www.geeksforgeeks.org/java/java-memory-management/)  
[인프런 강의](https://www.inflearn.com/course/%EB%8F%85%ED%95%98%EA%B2%8C-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-java-part2/dashboard)  
https://velog.io/@impala/JAVA-JVM-Runtime-Data-Area

