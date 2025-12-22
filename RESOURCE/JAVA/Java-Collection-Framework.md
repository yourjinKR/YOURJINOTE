
# 컬렉션 프레임워크

다수의 데이터를 쉽고 효과적으로 처리할 수 있는 표준화된 방법을 제공하는 클래스의 집합을 의미한다.  
[자료 구조](../CS/Data-Structure-자료-구조.md)와 알고리즘을 구조화하여 클래스로 구현해 놓은 것이다.

- 컬렉션 : 여러 객체(데이터)를 모아 놓은 것
- 프레임워크 : 표준화된 체계적인 프로그래밍 방식
- 컬렉션 클래스 : 다수의 데이터를 저장 할 수 있는 클래스


```java
public interface Collection<E> extends Iterable<E> {}
```

모든 `Collection`은 [Iterable](Java-Iterator.md)를 상속받고 있다.

<br>

# 핵심 인터페이스

- [List](Java-List.md) : 순서가 있는 데이터의 집합이며 중복을 허용한다.
- [Set](Java-Set.md) : 순서를 유지하기 않는 데이터의 집합, 데이터의 중복을 허용하지 않는다.
- Map : 키와 값의 쌍으로 이루어진 데이터의 집합이다.

![Pasted image 20251216133834](../../GALLERY/Pasted%20image%2020251216133834.png)

## Queue

자바에서 큐는 인터페이스이기에 별도의 클래스로 구현해야 된다.  
대표적으로 `LinkedList`로 구현한다.

```java
Queue<Integer> queue = new LinkedList<>();  
  
queue.offer(1);  
queue.offer(2);  
queue.offer(3);  
  
System.out.println(queue.poll());  
System.out.println(queue.poll());  
System.out.println(queue.poll());  
System.out.println(queue.poll()); // null
```

큐에 데이터가 없을 때 `poll()`를 호출 시 `null`를 반환한다.


<br>

# Collection 인터페이스의 메서드

컬렉션들이 공통적으로 사용하는 기능들을 추상화하여 관리한다.

### `Collection<E>`

| 메서드                                 | 반환 타입      | 설명           |
| ----------------------------------- | ---------- | ------------ |
| `add(E e)`                          | `boolean`  | 요소 추가        |
| `addAll(Collection<? extends E> c)` | `boolean`  | 컬렉션 전체 추가    |
| `remove(Object o)`                  | `boolean`  | 요소 제거        |
| `removeAll(Collection<?> c)`        | `boolean`  | 컬렉션 전체 제거    |
| `clear()`                           | `void`     | 모든 요소 제거     |
| `contains(Object o)`                | `boolean`  | 요소 포함 여부     |
| `containsAll(Collection<?> c)`      | `boolean`  | 컬렉션 포함 여부    |
| `size()`                            | `int`      | 요소 개수        |
| `isEmpty()`                         | `boolean`  | 비어있는지 여부     |
| `iterator()`                        | `Iterator` | 반복자 반환       |
| `toArray()`                         | `Object[]` | 배열로 변환       |
| `toArray(T[] a)`                    | `<T> T[]`  | 지정 타입 배열로 변환 |

<br>

# SequencedCollection 

`Collection` 인터페이스에는 순서에 대한 개념이 없었고, 이로 인해 순서가 있는 컬렉션을 매개변수로 받거나 반환할때 명확히 표현하기 어려웠다.  
이를 해결하기 위해 JDK 21 버전에 추가됐으며 기존 구현체의 공통 메서드를 인터페이스에 정의

```java
interface SequencedCollection<E> extends Collection<E> {
    SequencedCollection<E> reversed();
    // methods promoted from Deque
    void addFirst(E);
    void addLast(E);
    E getFirst();
    E getLast();
    E removeFirst();
    E removeLast();
}
```

순서가 있기에 first와 last에 대한 개념이 존재함.

> SequencedCollection의 개념에 대해 집중하기 보다는 기존의 Collection 프레임워크에  대한 특징을 이해하는 것에 집중하자.

<br>

# 출처

[geeksforgkeeks](https://www.geeksforgeeks.org/java/java-collection-tutorial/)  
[tcp-school](http://www.tcpschool.com/java/java_collectionFramework_concept)  
[티스토리 - Collection 인터페이스 대표 메서드](https://travelbeeee.tistory.com/474)  
[티스토리 - 자바의 Queue](https://coding-factory.tistory.com/602)
