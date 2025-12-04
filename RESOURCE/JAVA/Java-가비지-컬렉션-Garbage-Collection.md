# Garbage Collection란?
동적으로 할당된 메모리 영역 중 사용하지 않는 영역을 탐지하여 해제하는 기능
- 앞으로 사용되지 않는 객체의 메모리 = Garbage
- Garbage를 정해진 스켈줄에 의해 정리해주는 것 = Garbage Collection

# 자바의 Garbage Collector
사용되지 않는 객체를 메모리에서 자동으로 정리하는 역할을 합니다.<br>이로 인해 메모리 누수를 방지하고 자바 프로그램의 메모리 관리를 용이하게 합니다.

[힙](Java-Runtime-Data-Areas.md#힙-Heap) 영역에서 참조되지 않는 개체를 수집 및 제거

> **Heap**
> 동적으로 할당한 메모리 영역<br>모든 Object 타입의 데이터가 할당, Heap영역의 Object를 가리키는 참조 변수가 Stack에 할당

# Garbage Collector 알고리즘

버전별로 여러 GC가 있으며 각 GC마다 사용하는 알고리즘의 형태가 다르다.

## Reference Counting Algorithm
- Garbage 탐색에 초점을 맞춘 알고리즘
- 각 객체마다 Reference Count를 관리하며, 이 카운트가 0이 되면 GC 수행
- 카운트가 0이 되면 메모리에서 제거된다는 장점
- 순환 참조 구조에서 Reference Count가 0이 되지 않는 문제가 발생하여 Memory Leak 발생 가능

### 참고
![Pasted image 20251128172144](../../GALLERY/Pasted%20image%2020251128172144.png)
글로벌 루트(런타임 시발점)을 기준으로 자신을 레퍼런스를 하는 횟수를 기록하고 count가 0인 것은 삭제함

![Pasted image 20251128172328](../../GALLERY/Pasted%20image%2020251128172328.png)
순환참조시 Root가 참조하지 않았음에도 불구하고 count가 전부 1이기에 메모리를 해제할 수 없다.

## Mark And Sweep Algorithm
Reference Counting Algorithm의 단점을 해결하기 위해 나온 알고리즘이다.
### Mark and Sweep 과정
1. Stack의 모든 변수를 스캔하면서 각각 어떤 객체를 참조하고 있는지 찾아서 마킹한다. (Mark)
2. Reachable Object가 참조하고 있는 객체를 찾아서 마킹한다. (Mark)
3. 마킹되지 않은 객체를 Heap에서 제거한다. (Sweep)

![Pasted image 20251204202919](../../GALLERY/Pasted%20image%2020251204202919.png)
### 한계점
해당 방식은 메모리가 Fragmentation(파편화)되는 단점이 있다.

### 참고
![Pasted image 20251128172435](../../GALLERY/Pasted%20image%2020251128172435.png)
root가 참조하는 메모리를 순차적으로 접근하여 mark<br>mark가 안됐다면 sweep 상태로 둔다. 그리고 sweep 상태의 메모리는 삭제함.

![Pasted image 20251128172532](../../GALLERY/Pasted%20image%2020251128172532.png)

## Mark-Sweep-Compact Algorithm
기존의 Mark And Sweep Algorithm 방식에서 발생하는 **파편화 문제를 해결**하기 위한 방식이다.

Mark And Sweep Algorithm처럼 참조되는 객체들에 대해서 마크하고, 참조되지 않으면 삭제한다.<br>**그 후에 메모리를 정리**하여 메모리 파편화를 해결한다.

![Pasted image 20251204203236](../../GALLERY/Pasted%20image%2020251204203236.png)
### 한계점
**모든 객체를 새 위치에 복사**하고 해당 객체에 대한 모든 참조를 업데이트해야 하므로 **일시 중지 시간이 증가**하고 **OverHead가 발생**할 수 있다.<br>즉, STW 시간이 길어질 수 있다.
## Mark And Copy Algorithm
메모리 파편화를 해결하기 위해 제시된 알고리즘이다.<br>현재 GC가 사용하는 알고리즘은 해당 알고리즘을 발전시킨 형태

Heap을 Active 영역과 InActive 영역으로 나눈다.<br>Active 영역에만 객체를 할당하고, 가득차면 GC를 수행한다.<br>GC를 수행하면 Suspend 상태가 되고 살아남은 객체들은 InActive 영역으로 복사한다.

> 복사하는 동안 프로그램이 Suspend 상태이므로 Stop-The-Copying라고도 한다.

복사가 끝나면 Active 영역에는 Garbage만 남고 InActive 영역에는 살아남은 객체만 남는다.<br>이후 Garbage를 제거하면 Active 영역은 Free Memory 상태가 된다.<br>즉, Active 영역과 InActive 영역이 바뀌면서 정리가 되는 개념이다.

Active 영역과 InActive 영역은 논리적인 구분이다.

![Pasted image 20251204211455](../../GALLERY/Pasted%20image%2020251204211455.png)
### 한계점
메모리 파편화를 해결하기위해 나온것처럼 복사할 때 연속된 메모리 공간에 쌓게된다.<br>단점은 Heap의 절반만 사용가능하는 비효율성과 Suspend 현상, 복사할 때 Overhead가 존재한다.

## Generational Algorithm
Mark And Copy Algorithm은 많은 Overhead를 발생시킨다.<br>이에 대한 대안으로  Weak Generational Hypothesis 가설을 토대로 나온 알고리즘이다.

![Pasted image 20251204211536](../../GALLERY/Pasted%20image%2020251204211536.png)
객체는 Young Gen에 할당되고 GC가 수행될 때마다 살아남은 객체에 Age를 기록한다.<br>Age가 특정 임계값을 넘어서면 Old Gen으로 Copy하는 작업을 수행한다. (Promotion)

Old Gen으로 복사할 때 압축 Compaction(압축) 작업이 이루어지며 Old Gen이 가득 차서 Full GC를 발생하게 된다면 [STW](STW-Stop-The-World.md)가 발생한다.

해당 알고리즘은 메모리 파편화, 메모리 활용, Copy 알고리즘의 OverHead 등 여러 알고리즘의 단점을 극복했다.

# Garbage Collection의 종류

- Young 및 Old 를 위한 Serial GC (직렬)
- Young 및 Old 를 위한 Parallel GC (병렬)
- Young 을 위한 Parallel New + Old 를 위한 Concurrent Mark and Sweep (CMS)
- Young 와 Old 를 포함하는 G1

## Serial GC
- Java 5,6 기본값 GC
- Young Gen에는 Mark And Copy 알고리즘 사용
- Old Gen에는 Mark-Sweep-Compact 알고리즘 사용

Young과 Old가 연속적으로 처리되며 하나의 CPU를 사용한다.<br>이 처리를 수행할 때 STW가 발생한다.

즉, Single Thread Collector 이므로 **병렬화를 할 수 없고** 여러 CPU 코어를 활용할 수 없다.<br>JVM에서 CPU 코어를 하나만 사용하기 때문에 단일 CPU가 있는 환경에서 실행되는 수백MB의 힙 크기 JVM에서만 권장된다.

## Parallel GC (Throughput GC)
- Java 7,8 기본값 GC
- Young Gen에는 Mark And Copy 알고리즘 사용
- Old Gen에는 Mark-Sweep-Compact 알고리즘 사용

이 방식의 목표는 다른 **CPU가 대기 상태로 남아있는 것을 최소화**하는 것이다.<br>Serial GC와 달리 **두 GC를 병렬로 처리**한다.<br>많은 CPU를 사용하기에 GC 부하를 줄이고 애플리케이션 처리량을 증가시킨다.

멀티 코어 시스템에 적합하다.

## Concurrent Mark and Sweep (CMS)
- Java9부터 Deprecated
- Young Gen에는 Paralle STW Mark And Copy 알고리즘 사용
- Old Gen에는 주로 Concurrent Mark-Sweep 알고리즘을 사용

Old 에서 GC하는동안 **긴 일시 중지를 방지**하도록 설계

- Old Gen을 **압축하지 않고 여유 목록(Free-Lists)을 사용**하여 회수된 공간을 관리
- 응용 프로그램과 동시에 Mark And Sweep 단계에서 대부분의 작업 수행

이런 단계를 수행하기 위해 애플리케이션 **쓰레드를 명시적으로 중지하지 않는다.**<br>그래도 여전히 응용 프로그램 쓰레드와 CPU 시간을 놓고 경쟁한다는 점에 유의해야 한다.<br>기본적으로 이 GC 알고리즘에서 사용하는 쓰레드 수는 컴퓨터 물리적 코어 수의 1/4와 같다.

압축을 수행하지 않기에 메모리 파편화가 발생한다.<br>이로 인해 메모리 할당이 불가능해지면 Concurrent Mode Failure 경고가 발생하면서 Compaction 작업을 수행한다.<br>이 때는 다른 STW 시간보다 긴 시간이 소요되므로 서버 환경에서 위험할 수 있다.

## Garbage First(G1) GC
- Java 9부터 기본값 GC
- Young Gen에는 Snapshot-At-The-Beginning(SATB)
- Old Gen에는 Snapshot-At-The-Beginning(SATB)

대규모 Heap 사이즈와 멀티 프로세서 환경에서 사용되도록 만들어진 GC이다.

메모리를 페이징 하듯이 **논리적인 단위(Region)으로 Heap 전체 영역을 나눠서 관리**한다.<br>Region 단위로 메모리를 관리하여 Compaction 단계를 진행하고 메모리 파편화 문제를 해결했다. 
또한 STW 시간을 예측할 수 있다.

![](https://blog.kakaocdn.net/dna/bJUMr3/btrODeWxtnR/AAAAAAAAAAAAAAAAAAAAAJa7vIIsLLjEA-_vK6VBRQLht0ccC3FUH3O7F7Aicc33/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1767193199&allow_ip=&allow_referer=&signature=yn1Mq4NKbk7SaR5wt91JTW56tXU%3D)

위 그림처럼 Region 영역에 객체를 할당하고 GC를 실행한다.

해당 영역이 꽉 차면 다른 영역에서 객체를 할당하고 GC를 실행한다.<br>그래서 지금까지 Young -> Old로 이동하는 단계가 아닌 GC 방식이라 이해하면 된다.

# 출처
[유튜브-[개발면접3분] Garbage Collection: Mark and Sweep](https://www.youtube.com/watch?v=X2Hcos6iSYw)