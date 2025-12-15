# 세대 단위 컬렉션 이론 (Generational collection theory)

- 약한 세대 가설 : 대다수 객체는 일찍 죽는다.
- 강한 세대 가설 : 가비지 컬렉션 과정에서 살아남은 횟수가 늘어날수록 더 오래 살 가능성이 커진다.
- 세대 간 참조 가설 : 세대 간 참조의 개수는 같은 세대 안에서 참조보다 훨씬 적다.

# 영역

Heap 영역은 효율적인 Garbage Collection을 위해 크게 3가지 영역으로 구분된다.

- Young Generation
- Old Generation
- Permanent (Metaspace)

![Pasted image 20251204204444](../../GALLERY/Pasted%20image%2020251204204444.png)


## Young Generation

새롭게 생성된 객체가 할당되는 영역  
대부분 객체가 금방 Unreachable한 상태가 되기 때문에 많은 객체가 Young 영역에 생성되었다가 사라짐.  
Young 영역에 대한 가비지 컬렉션을 Minor GC라고 부른다.

### Eden

- 객체 생성 직후 저장되는 영역
- Minor GC 발생 시 Survivor 영역으로 이동
- Copy & Scavenge 알고리즘

### Survivor 0, 1

- Minor GC 발생 시 Eden, S0에서 살아남은 객체는 S1으로 이동
- S1에서 살아남은 객체는 Old 영역으로 이동
- age bit 사용 (참조 계수)

### Minor GC

- `new` 키워드를 통해 새로운 인스턴스가 생성되면 Eden영역에 저장
- 이후 Survivor로 이동
- 시간이 지나면서 이 영역에 있는 데이터는 우선순위에 따라 Old 영역으로 이동 혹은 GC에 의해 수거

## Tenured(Old) Generation

Young Generation영역에서 소멸하지 않고 남아있던 인스턴스가 이동되어 저장되는 영역.  
**Young 영역보다 크게 할당**되며, 영역의 크기가 큰 만큼 가비지는 적게 발생한다.  
Old 영역에 대한 가비지 컬렉션을 Major GC 또는 Full GC라고 부른다.

### Major GC (Full GC)

- Young 영역에서 살아남은 객체가 Old 영역에 복사
- Old 영역에 할당된 메모리가 허용치를 넘게 되면, Old 영역에 있는 모든 인스턴스들을 검사
- 참조되지 않는 인스턴스를 한꺼번에 삭제
- Major GC가 발생하면 [STW](STW-Stop-The-World.md) 발생

## Permanent Generation

ClassLoader에 의해 동적으로 로딩된 클래스의 메타데이터가 저장되는 영역  
이 정보들은 JVM 실행 도중에 변경되지 않으며, JVM 종료 시까지 유지

> Java 8 이후 **Metaspace로 대체**되어 **Heap영역에서 제외**되었다.

## Metaspace

- Perm 영역에서 저장하던 **클래스의 메타 정보**들이 이 영역에서 저장
- Native Memory 영역에 위치하며 JVM이 아닌 **운영체제가 관리**
- 클래스 메타데이터와 **리플렉션**을 이용하는 애플리케이션에서 사용하는 일부 메모리를 저장

# Perm 영역을 대체한 이유

PermGem은 JVM 내부에 고정된 크기의 메모리 공간이었으며 아래와 같은 문제가 자주 발생한다.

- 클래스를 과도하게 많이 로딩
- 동적 프록시를 자주 생성
- 재시작 없이 애플리케이션을 여러 번 redeploy하는 경우
- 적절한 크기를 조절하기 힘들었음
	- 너무 적게 잡으면 OOM 발생
	- 너무 크게 잡으면 메모리 낭비
- Perm 영역의 메모리 누수, OutOfMemoryError 등과 같은 문제
    - 클래스 로딩 및 언로딩 과정에서 메모리 할당 및 해제의 빈번한 발생으로 인해 이러한 문제가 더 심각해졌다.
- 클래스 메타데이터를 Native Memory에 저장하면서,  JVM에서의 `OutOfMemoryError` 문제가 해결되었다.