# JVM (Java Virtual Machine)
- 자바 애플리케이션을 실행하는 [가상-머신-Virtual-Machine](%EA%B0%80%EC%83%81-%EB%A8%B8%EC%8B%A0-Virtual-Machine.md)
- 운영채제와 상관 없이 하나의 코드로 실행
- 대표적인 단점으로는 속도가 느림
	- [JIT 컴파일러](Java-JIT-컴파일러.md)로 개선은 되었음
- Java Compiler가 JAVA로 작성된 소스 코드 .java 파일을 .class 파일인 **Byte Code**로 컴파일한다.  
- 이제 이 **Byte Code**를 기계어로 변환시키기 위해 가상 CPU가 필요한데, 이것이 **JVM(Java Virtual Machine)**의 역할이다.
- JVM이 **Byte Code**를 **기계어(Binary Code)**로 변환한다.
- 이렇게 JVM에 의해 컴파일된 기계어는 바로 CPU에서 실행되어 사용자에게 서비스를 제공해준다

## 프로그램 실행 과정

![java-jvm](https://blog.kakaocdn.net/dna/sd2Hq/btrII1Qq6il/AAAAAAAAAAAAAAAAAAAAADdp4tGiLv-Dj-Gy6Cw5mdkfrK6ARs6aoFj7iQ7F22-h/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1764514799&allow_ip=&allow_referer=&signature=0lQVjYwnisLuCNCqOv2Hotk5rDI%3D)

1. 자바 소스 코드(.java 파일)를 작성
2. JDK의 **컴파일러**가 자바 소스 파일을 **바이트코드**(.class 파일)로 변환
3. JVM의 **클래스 로더**가 바이트코드를 메모리에 로드합니다.
4. JVM의 **실행 엔진**이 바이트코드를 운영체제에 맞는 기계어로 변환하고 실행
5. **가비지 컬렉터**가 메모리 관리를 자동으로 수행

## 구조
![Pasted image 20251202202142](Pasted%20image%2020251202202142.png)

## Class Loader
• JVM의 **Class Loader**는 자바 바이트코드를 JVM 내부로 로드하는 역할을 수행하고 **여러 클래스를 메모리에 적재**하고, 필요한 **클래스를 동적으로 로드**함
• 클래스 로딩 과정은 아래와 같이 크게 세 단계로 분류
### 로딩
클래스 파일을 찾아서 메모리에 로드
### 연결
클래스 파일 간의 의존성을 해소하고 심볼릭 레퍼런스를 메모리 주소로 변경
### 초기화
정적 필드 및 초기화 블록을 실행

[자세히](Java-클래스-로더-Class-Loader.md)

## 메모리 구조 (Runtime Data Areas)
JVM이 프로그램을 실행할 때, 메모리는 **여러 영역**으로 나뉘어 관리
### 메소드 영역(Method Area)
모든 클래스의 메타데이터(클래스 구조, 메서드 정보 등)를 저장
### 힙(Heap)
객체가 동적으로 할당되는 영역으로, 자바에서 생성된 모든 객체는 힙 메모리에 저장
### 스택(Stack)
각 쓰레드마다 별도로 할당되는 메모리 영역, 메서드 호출 시 로컬 변수를 저장, 관리
### PC 레지스터(Program Counter Register)
현재 실행 중인 명령어의 위치를 저장합니다.
### 네이티브 메소드 스택(Native Method Stack)
자바 외부의 네이티브 메소드(C, C++ 등)를 호출

[자세히](Java-Runtime-Data-Areas.md)

## Execution Engine (실행 엔진)
• 실행 엔진은 JVM에서 바이트코드를 실제 기계어로 변환하고 실행하는 핵심 요소입니다. 이 과정에서 여러 최적화 기술이 사용됩니다.
• 인터프리터(Interpreter): 바이트코드를 한 줄씩 읽어 즉시 기계어로 변환하여 실행하는 방식입니다. 속도가 느리지만, 구현이 간단합니다.
• JIT(Just-In-Time) 컴파일러: 실행 중인 코드를 분석하여, 자주 사용되는 코드(핫스팟)를 기계어로 변환하고 캐시하는 방식입니다. 한번 컴파일된 코드는 다시 변환할 필요가 없기 때문에 속도가 매우 빠릅니다.
• **[가비지 컬렉션(Garbage Collection)](Java-가비지%20컬렉션-Garbage-Collection.md)**: 사용되지 않는 객체를 메모리에서 자동으로 정리하는 역할을 합니다. 이로 인해 메모리 누수를 방지하고 자바 프로그램의 메모리 관리를 용이하게 합니다.
## Java Native Interface
• 자바는 순수 자바 코드 외에도 C나 C++과 같은 네이티브 코드를 호출할 수 있도록 
**JNI**를 제공합니다. 이를 통해 성능이 중요한 부분에서는 네이티브 코드를 사용할 수 있습니다.

# 출처
[얄팍한 코딩 사전 유튜브](https://www.youtube.com/watch?v=OxvtGYvVkRU)
[블로그](https://inpa.tistory.com/entry/JAVA-%E2%98%95-JDK-JRE-JVM-%EA%B0%9C%EB%85%90-%EA%B5%AC%EC%84%B1-%EC%9B%90%EB%A6%AC-%F0%9F%92%AF-%EC%99%84%EB%B2%BD-%EC%B4%9D%EC%A0%95%EB%A6%AC)
[블로그2](https://sung-98.tistory.com/133)