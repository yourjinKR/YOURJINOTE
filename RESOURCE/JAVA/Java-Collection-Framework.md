
# 컬렉션 프레임워크

[자바](JAVA.md)에서 다수의 **데이터를 쉽고 효과적으로 처리할 수 있는 표준화된 방법을 제공**하는 클래스의 집합을 의미한다. [자료 구조](../CS/Data-Structure-자료-구조.md)와 알고리즘을 구조화하여 클래스로 구현해 놓은 것이다.

- 컬렉션 : 여러 객체(데이터)를 모아 놓은 것
- 프레임워크 : 표준화된 체계적인 프로그래밍 방식
- 컬렉션 클래스 : 다수의 데이터를 저장 할 수 있는 클래스


```java
public interface Collection<E> extends Iterable<E> {}
```

- 모든 `Collection`은 [Iterable](Java-Iterator.md)를 상속받고 있다.

> `iterable`은 객체가 `iteror`를 제공하여 `for-each` 문으로 순회될 수 있음을 보장하는 인터페이스 

- [Collections](Java-Collections.md)클래스를 통해 여러 유틸성 메서드를 제공한다.

<br>

# 핵심 인터페이스

- [List](Java-List.md) : 순서가 있는 데이터의 집합이며 중복을 허용한다.
- [Set](Java-Set.md) : 순서를 유지하기 않는 데이터의 집합, 데이터의 중복을 허용하지 않는다.
- [Map](Java-Map.md) : 키와 값의 쌍으로 이루어진 데이터의 집합이다.

각각의 인터페이스의 [차이점](../../AREA/취업/기술면접/자바의%20List%20Set%20Map의%20차이를%20설명해주세요.md)은 **순서 유지와 중복 허용 여부**에 따라 구분된다. 

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

# AbstractCollection

`Collection` 인터페이스의 기본 구현을 제공하여 해당 인터페이스를 구현하는데 부담을 줄인다.  
해당 구조는 [템플릿 메소드 패턴](../CS/Template-Method-Pattern-템플릿-메소드-패턴.md)의 전형적인 예시이다.

- 구현 부담 감소
- 일관된 기본 동작 제공
- 확장에 유리한 구조

대표적인 예시는 다음과 같다.

- `AbstractCollection`
- `AbstractList`
- `AbstractSet`
- `AbstractMap`
- `AbstractQueue`

수정 가능한 컬렉션을 구현하려면 해당 클래스의 `add()`를 추가로 재정의가 필요함.

```java
public boolean add(E e) {  
    throw new UnsupportedOperationException();  
}
```

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
[oracle - AbstractCollection](https://docs.oracle.com/javase/8/docs/api/java/util/AbstractCollection.html)  
[tcp-school](http://www.tcpschool.com/java/java_collectionFramework_concept)  
[티스토리 - Collection 인터페이스 대표 메서드](https://travelbeeee.tistory.com/474)  
[티스토리 - 자바의 Queue](https://coding-factory.tistory.com/602)

<br>

# 다시 점검할 부분

[기술면접-Java-Collection-Framework](../../AREA/취업/기술면접/기술면접-Java-Collection-Framework.md)  