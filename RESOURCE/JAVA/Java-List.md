
# List Interface

List 인터페이스는 [Collection 인터페이스](Java-Collection-Framework.md)를 상속하며 java.util 패키지에 속한다.  
중복이 허용되고 인덱스를 통해 요소에 접근할 수 있는 순서가 지정된 컬렉션을 저장하는 데 사용됩니다.

- 삽입 순서를 유지
- 중복 요소를 허용
- `null` 요소를 지원
- `ListIterator`를 사용하여 양방향 순회를 지원

```java
public interface List<E> extends Collection<E> {}
```

<br>

# List Interface 구현체

리스트를 사용시 아래와 같이 해당 인터페이스를 구현하는 클래스를 인스턴스화 한다.

```java
List<Type> list = new ArrayList<Type>();
```

![Pasted image 20251218131710](../../GALLERY/Pasted%20image%2020251218131710.png)

- ArrayList
- LinkedList
- Vector
- Stack

## ArrayList

- 내부적으로 `elementData` 라는 `Object` **배열**에 값을 관리
- 구조가 간단하며 임의의 요소에 접근하는 시간이 짧다 (인덱스)
- 비순차적 요소 삽입/삭제 성능이 좋지 않음 (배열의 크기를 조정하기 때문에)

```java
public class ArrayList<E> extends AbstractList<E>  
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable  
{
	transient Object[] elementData;
}
```

## LinkedList

- 이중 연결 리스트를 사용하여 구현, 각 요소(`Node`)가 이전 요소와 다음 요소의 참조를 가짐
- 요소 추가/삭제 성능에 유리 (배열의 크기를 조정할 필요가 없기에)
- 임의의 요소 접근시 오래 걸린다 (각 요소를 순차적으로 탐색)

```java
public class LinkedList<E>  
    extends AbstractSequentialList<E>  
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable  
{

    /**
     * Pointer to first node.
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     */
    transient Node<E> last;

	private static class Node<E> {  
	    E item;  
	    Node<E> next;  
	    Node<E> prev;  // 이중 연결 리스트, 이전 노드에 대한 참조 또한 존재 
	  
	    Node(Node<E> prev, E element, Node<E> next) {  
	        this.item = element;  
	        this.next = next;  
	        this.prev = prev;  
	    }  
	}
}
```

> **이중 연결 리스트란?**  
> 링크드 리스트는 이동방향이 단방향이기에 다음 요소에 대한 접근은 쉽지만, 이전 요소에 대한 접근은 어웠다.  
> 이를 해결하기 위해 이전 요소에 대한 참조를 추가하여 양방향 이동을 지원하는 리스트 형식이다.

### Linked List 종류

- Circular linked list : 꼬리와 헤드가 연결된 리스트
- Doubly linked list : 노드 간 양방향으로 연결된 리스트
- Circular doubly linked list : Circular linked list + Doubly linked list

## ArrayList vs LinkedList 비교

| 연산                                  | ArrayList                           | LinkedList                     |
| ----------------------------------- | ----------------------------------- | ------------------------------ |
| 구현                                  | 배열 사용                               | 노드를 연결                         |
| 데이터 접근  시간                          | 모든 뎅이터 상수 시간 접근                     | 위치에 따라 이동 시간 발생                |
| 삽입/삭제 시간                            | 삽입/삭제 시 데이터 시프트가 필요한 경우 추가 시간 발생    | 삽입/삭제 위치에 따라 그 위치까지 이동하는 시간 발생 |
| 리스트 크기 확장                           | 배열 확장이 필요할 경우 새로운 배열에 복사하는 추가 시간 발생 | -                              |
| 검색                                  | 최악의 경우 리스트에 있는 아이템 수 만큼 확인          | 최악의 경우 리스트에 있는 아이템 수 만큼 확인     |
| [CPU chache](../CS/CPU%20chache.md) | CPU cache 이점 활용                     | -                              |

> `ArrayList`는 내부적으로 연속적인 배열 구조를 사용하기 때문에 공간 지역성(spatial locality)이 적용되어 CPU 캐시 히트율이 높아지며, 순차 접근 시 매우 빠른 성능을 보인다.


**ArrayList**
- 배열 사용
- 순차적인 요소 삽입/삭제
- 임의의 요소 조회

**LinkedList**
- 임의의 요소 삽입/삭제
- 순차적인 조회

> 데이터가 불변일때는 `ArrayList`가 압도적으로 유리한 성능을 제공한다.

```java
List<?> list1 = new ArrayList<>();  
List<?> list2 = new LinkedList<>();  
  
ArrayList<?> list3 = new ArrayList<>(list2);  
LinkedList<?> list4 = new LinkedList<>(list2);
```

서로 변환이 가능한 생성자를 제공하기에 연산 과정을 고려하여 유리한 자료 구조를 선택하여 프로그램을 더욱 효율적으로 관리할 수 있다.

### 성능 정리

| 연산              | ArrayList | LinkedList |
| --------------- | --------- | ---------- |
| get(index)      | O(1)      | O(n)       |
| add(e) 끝에 추가    | O(1)*     | O(1)       |
| add(index, e)   | O(n)      | O(n)       |
| remove(index)   | O(n)      | O(n)       |
| Iterator remove | O(n)      | O(1)       |

## Vector / Stack (Legacy)

### Vector

- `ArrayList`와 동일한 동적 배열 구조
- 모든 메서드가 `synchronized` 명시되어 있어 성능 저하
- 현업에서는 거의 사용하지 않음
### Stack

- `Vector`를 상속한 구식 구조
- 대신 `Deque` 기반의 `ArrayDeque` 사용을 권장한다.
- `push()`, `pop()`을 통해 데이터를 삽입/삭제 한다.
	- 빈 Stack `pop()`을 요청하면 `EmptyStackException`이 발생

```java
Stack<Integer> stack =  new Stack<>();  
  
stack.push(1);  
stack.push(2);  
stack.push(3);  
  
System.out.println(stack.pop()); // 3  
System.out.println(stack.pop()); // 2  
System.out.println(stack.pop()); // 1  
System.out.println(stack.pop()); // EmptyStackException
```

# 주의사항

### `remove()`

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.remove(1);    // 인덱스 1 삭제
list.remove(Integer.valueOf(1)); // 값 1 삭제
```

<br>

# 출처

[f-lab](https://f-lab.kr/insight/java-arraylist-vs-linkedlist-20240521)  
[쉬운코드-리스트를 설명합니다](https://www.youtube.com/watch?v=xvi-n11kym0)

