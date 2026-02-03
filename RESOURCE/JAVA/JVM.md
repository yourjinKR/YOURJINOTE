# JVM (Java Virtual Machine)

JRE의 핵심 구성 요소로, [자바](JAVA.md) 애플리케이션을 실행하는 [가상 머신](../CS/가상-머신-Virtual-Machine.md)

- JVM이 **Byte Code**를 **기계어(Binary Code)** 로 변환한다.
- 자바 바이트코드와 하드웨어 사이에서 인터프리터 역할을 수행
- 운영체제와 상관 없이 하나의 코드로 실행
- 자바 프로그램이 메모리를 할당하고 해제하는 방법을 제어

> 자바 컴파일러가 소스코드(.java)를 바이트 코드(.class)로 컴파일한다.  
> 해당 바이트 코드를 기계어로 변환하기 위해 사용되는 가상 CPU가 JVM이다.  

- 대표적인 단점으로는 **속도가 느림**
	- [JIT 컴파일러](Java-JIT-컴파일러.md)로 개선은 되었음

## 프로그램 실행 과정

![Pasted image 20251214221502](../../GALLERY/Pasted%20image%2020251214221502.png)

1. 자바 **소스 코드**(.java 파일)를 작성  
2. **JDK**의 **컴파일러**가 자바 소스 파일을 **바이트코드**(.class 파일)로 변환  
3. **JVM**의 **클래스 로더**가 바이트코드를 메모리에 로드합니다.  
4. **JVM**의 **실행 엔진**이 바이트코드를 운영체제에 맞는 기계어로 변환하고 실행  
5. **가비지 컬렉터**가 **메모리 관리**를 자동으로 수행  

## 구조

![Pasted image 20251202202142](../../GALLERY/Pasted%20image%2020251202202142.png)

## Class Loader

• JVM의 [Class Loader](Java-클래스-로더-Class-Loader.md)는 자바 바이트코드를 JVM 내부로 로드하는 역할을 수행하고 **여러 클래스를 메모리에 적재**하고, 필요한 **클래스를 동적으로 로드**한다.

## Runtime Data Areas

JVM은 메모리를 여러 [런타임 데이터 영역](Java-Runtime-Data-Areas.md)으로 나눈다.  
일부 영역은 JVM 전체에서 공통으로 사용되고 다른 영역은 각 스레드마다 별도로 생성된다.

- 메서드 영역 (Method Area)
- 힙 영역 (Heap Area)
- 스택 영역 (Stack Area)
- PC 레지스터 (Program Counter Register)
- 네이티브 메서드 스택 (Native Method Stack)

## Execution Engine (실행 엔진)

실행 엔진은 JVM에서 바이트코드를 실제 기계어로 변환하고 실행하는 핵심 요소이다.

### Interpreter

인터프리터는 바이트코드를 한 줄씩 읽어 즉시 기계어로 변환하여 실행하는 방식입니다. 속도가 느리지만, 구현이 간단합니다.

### JIT Compiler

[JIT Compiler](Java-JIT-컴파일러.md)는 실행 중인 코드를 분석하여, 자주 사용되는 코드(핫스팟)를 기계어로 변환하고 캐시하는 방식입니다. 한번 컴파일된 코드는 다시 변환할 필요가 없기 때문에 속도가 매우 빠릅니다.

### Garbage Collection

[가비지 컬렉션](Java-가비지-컬렉션-Garbage-Collection.md)은사용되지 않는 객체를 메모리에서 자동으로 정리하는 역할을 합니다. 이로 인해 메모리 누수를 방지하고 자바 프로그램의 메모리 관리를 용이하게 합니다.

## Java Native Interface (JNI)

자바는 순수 자바 코드 외에도 C나 C++과 같은 네이티브 코드를 호출할 수 있도록 **JNI**를 제공합니다.  
이를 통해 성능이 중요한 부분에서는 네이티브 코드를 사용할 수 있습니다.

# 출처
[geeksforgeeks](https://www.geeksforgeeks.org/java/how-jvm-works-jvm-architecture/)  
[얄팍한 코딩 사전 유튜브](https://www.youtube.com/watch?v=OxvtGYvVkRU)  
[블로그](https://inpa.tistory.com/entry/JAVA-%E2%98%95-JDK-JRE-JVM-%EA%B0%9C%EB%85%90-%EA%B5%AC%EC%84%B1-%EC%9B%90%EB%A6%AC-%F0%9F%92%AF-%EC%99%84%EB%B2%BD-%EC%B4%9D%EC%A0%95%EB%A6%AC)  
[블로그2](https://sung-98.tistory.com/133)  
[velog - Java 실행 과정 및 JVM](https://velog.io/@baekgom/JVMJava-Virtual-Machine-%EC%9E%90%EB%B0%94-%EA%B0%80%EC%83%81-%EB%A8%B8%EC%8B%A0#3-%EB%B3%80%ED%99%98-%EC%9D%B4%EC%9C%A0)

