# Garbage Collection란?
동적으로 할당된 메모리 영역 중 사용하지 않는 영역을 탐지하여 해제하는 기능
# 자바의 Garbage Collector
사용되지 않는 객체를 메모리에서 자동으로 정리하는 역할을 합니다. 
이로 인해 메모리 누수를 방지하고 자바 프로그램의 메모리 관리를 용이하게 합니다.

[[RESOURCE/JAVA/[Java] 가비지 컬렉션 (Garbage Collection).md|힙]] 영역에서 참조되지 않는 개체를 수집 및 제거

> **Heap**
> 동적으로 할당한 메모리 영역
> 모든 Object 타입의 데이터가 할당, Heap영역의 Object를 가리키는 참조 변수가 Stack에 할당

> GC수행 시 프로그램이 일시적으로 멈추는데 이걸 **stop-the-world**라고 부른다.

## 기존 방식 알고리즘 (Reference Count)
![[Pasted image 20251128172144.png]]
글로벌 루트(런타임 시발점)을 기준으로 자신을 레퍼런스를 하는 횟수를 기록하고 count가 0인 것은 삭제함

![[Pasted image 20251128172328.png]]
순환참조시 Root가 참조하지 않았음에도 불구하고 count가 전부 1이기에 메모리를 해제할 수 없다.
## 자바의 알고리즘 (Mark and Sweep)
![[Pasted image 20251128172435.png]]
root가 참조하는 메모리를 순차적으로 접근하여 mark
mark가 안됐다면 sweep 상태로 둔다. 그리고 sweep 상태의 메모리는 삭제함.

![[Pasted image 20251128172532.png]]
### Mark and Sweep 과정
1. Stack의 모든 변수를 스캔하면서 각각 어떤 객체를 참조하고 있는지 찾아서 마킹한다. (Mark)
2. Reachable Object가 참조하고 있는 객체를 찾아서 마킹한다. (Mark)
3. 마킹되지 않은 객체를 Heap에서 제거한다. (Mark)
# 출처
[유튜브-[개발면접3분] Garbage Collection: Mark and Sweep](https://www.youtube.com/watch?v=X2Hcos6iSYw)