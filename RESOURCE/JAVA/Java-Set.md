# Set

- 중복 요소를 허용하지 않음
- TreeSet 구현을 제회하고는 최대 하나의 `null` 값만 포함, TreeSet은 `null` 값을 허용하지 않음
- 효율적인 검색, 삽입 및 삭제 작업을 제공

`add()`할때마다 중복을 검사하기에 반환 타입은 boolean이다.

> 이때 `equals()`와 `hashCode()`를 호출하기에 오버라이딩에 주의하자

##  계층 구조

- [HashSet](Java-HashSet.md) : 대용량 데이터 저장/탐색에 유리
- [EnumSet](Java-EnumSet.md) : 열거형 데이터를 다룰때
- [LinkedHashSet](Java-LinkedHashSet.md) : 순서를 저장함
- [TreeSet](Java-TreeSet.md) : 범위 탐색과 정렬에 유리

![Pasted image 20251222164652](../../GALLERY/Pasted%20image%2020251222164652.png)

## 메서드

|메서드|설명|
|---|---|
|`add(element)`|요소가 이미 없으면 추가하고, 추가되면 `true` 반환한다.|
|`addAll(collection)`|주어진 컬렉션의 모든 요소를 Set에 추가한다.|
|`clear()`|Set의 모든 요소를 제거한다.|
|`contains(element)`|특정 요소가 포함되어 있는지 확인한다.|
|`containsAll(collection)`|컬렉션의 모든 요소가 Set에 포함되어 있는지 확인한다.|
|`hashCode()`|Set의 해시 코드를 반환한다.|
|`isEmpty()`|Set이 비어 있는지 여부를 확인한다.|
|`iterator()`|Set을 순회할 수 있는 Iterator를 반환한다.|
|`remove(element)`|지정된 요소를 제거한다. 존재하면 제거 후 `true` 반환한다.|
|`removeAll(collection)`|Set에서 주어진 컬렉션에 포함된 모든 요소를 제거한다.|
|`retainAll(collection)`|주어진 컬렉션에 포함된 요소만 남기고 나머지는 제거한다.|
|`size()`|Set에 저장된 요소의 개수를 반환한다.|
|`toArray()`|Set 요소들을 동일한 요소로 구성된 배열로 반환한다.|

<br>

# 출처

[geeksforgeeks](https://www.geeksforgeeks.org/java/set-in-java/)

