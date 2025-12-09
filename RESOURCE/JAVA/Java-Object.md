# 자바에서 객체
자바에서는 기본형을 제외한 모든 데이터를 [객체](../CS/객체-Objcect.md)로 관리한다.  
뿐만 아니라 자바의 모든 클래스는 [Object 클래스](../JAVA/Java-Object-Class.md)를 상속받는다.

# Java Ordinary Object Pointer

객체를 가리키기 위한 포인터를 말한다.

- `instanceOop` : 단일 객체를 표현하기 위한 OOP
- `arrayOop` : 배열을 가리키기 위한 OOP

이 OOP들은 모두 `oopDesc`라는 클래스를 기반으로 한다.  
`oopDesc`는 `mark word`와 `klass word` 필드를 포함하고 있다.

- `mark word` : Object Header 정보를 포함
- `klass word` : Langauage Level에서 메타 데이터를 포함

> 이렇기에 자바의 모든 객체들은 기본적으로 12byte의 오버헤드를 가지고 있다.

# Java Object Layout

```
+----------------------+
| Mark Word            | 8 bytes (64-bit JVM)
+----------------------+
| Klass Pointer        | 4 or 8 bytes
+----------------------+
| Instance Fields      | (필드 크기 + 정렬 패딩)
+----------------------+
| Padding (8-byte align)
+----------------------+
```

- Object Header
- Instance Data
- Padding

## Object Header

- Mark word
- Klass word
- Length

### Mark word
Object Header를 설명  
각 **객체의 메타 데이터**가 저장되는 영역
#### Hash code
객체의 메모리 주소를 가리키어 각 객체 별로 고유하게 할당  
객체 생성 초기에는 0으로 존재하다가 `Object.hashcode()` 메소드가 호출되는 시점에 계산  
#### Object age
GC에서 살아남은 횟수를 기록  
#### Lock flag
객체를 중심으로 멀티 스레드 환경에서 경쟁 조건이 발생하는 문제를 해결하기 위한 것

![Pasted image 20251208122802](../../GALLERY/Pasted%20image%2020251208122802.png)
### Klass word

해당 객체의 **클래스 메타 데이터**가 존재하는 메모리 주소를 가리킨다. (포인터 역할을 수행)  
클래스에 종속적이라 모든 객체마다 동일한 메타데이터는 하나로 관리하면 되기 때문에, 이를 클래스 공유 메타데이터라고도 한다.

- 클래스 정보 : 해당 객체가 어떤 클래스의 인스턴스인지 나타냄
- VTable (Virtual Method Table) 포인터 : 가상 메서드 테이블 주소
	- 다형성을 지원하기 위한 메서드 테이블
- 필드 정보 : 필드의 오프셋, 타입 정보
- 정적 변수 (Static Fields) : 해당 클래스의 정적 필드에 대한 정보

Klass word는 모든 클래스가 공유하는 메타데이터 정보이기 때문에 [JVM Metaspace](Java-Runtime-Data-Areas.md#Metaspace) 라는 별도의 영역에서 저장 및 관리된다.

> **Mark word와 Klass word의 저장영역 비교**  
> 마크 워드는 메타데이터로 힙 내부에 존재하지만,  
> 클래스 워드는 클래스 정보를 가리키는 포인터이기에 최종적으로 메타스페이스 영역을 향한다.

### Length

자바에서는 배열의 경우 배열 길이도 객체 헤더에 저장한다.  
배열 길이를 알아야 배열 객체가 차지하는 메모리 크기를 계산할 수 있기 때문이다.
해당 객체가 배열이 아니라면 해당 정보는 존재하지 않는다.

> 따라서 객체 해더의 길이는 배열 유무에 따라 달라질 수 있다.

## Instance Data

객체가 실제로 담고 있는 멤버 변수들을 저장  
배치 방식은 아래와 같은 순서로 하여 정렬 비용을 최소화

```
long, double → int, float → short, char → byte, boolean → reference
```

## Padding

> CPU에서는 메모리 접근을 최적화하기 위해 WORD 단위로 접근한다.  
> 64bit 운영체제에서는 WORD가 8byte이기 때문에 이에 맞추어 메모리 크기를 8의 배수로 고정한다.  
> 메모리 공간 측면에서는 비효율적이나 CPU의 메모리 접근은 최적화된다.


# 출처

[인프런-널널한-개발자](https://www.inflearn.com/course/%EB%8F%85%ED%95%98%EA%B2%8C-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-java-part2/dashboard)

[baeldung-java-memory-layout](https://www.baeldung.com/java-memory-layout)

[JVM 내부의 힙 객체 헤더](https://mangkyu.tistory.com/448)

[Java Object의 Memory Layout과 Trino의 Slice](https://leeyh0216.github.io/posts/trino-slice/)

