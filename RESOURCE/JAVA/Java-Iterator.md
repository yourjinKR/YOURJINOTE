
# Iterator

컬렉션에 저장된 요소들을 순차적으로 순회하기 위해 표준화된 인터페이스이다.

```java
public interface Iterator<E> {
    boolean hasNext();
    E next();
    
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
    
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```

| 메서드                                                         | 설명                       |
| ----------------------------------------------------------- | ------------------------ |
| `boolean hasNext()`                                         | 다음 데이터가 있는지 확인           |
| `E next()`                                                  | 다음 요소를 반환하고 커서를 이동       |
| `default void remove()`                                     | 현재 요소 삭제                 |
| `default void forEachRemaining(Consumer<? super E> action)` | 남은 모든 요소에 반복 처리 (Java 8) |

<br>

## Iterable

```java
public interface Iterable<T> {  
    Iterator<T> iterator();
 }
```

- `Collection` 인터페이스는 `Iterator`를 상속 받음
- `Iterable`은 각 컬렉션의 특징에 맞는`Iterator` 객체를 반환

<br>

```java
Iterator<?> iterator = list1.iterator();  
  
while (iterator.hasNext()) {  
    System.out.println(iterator.next());  
}
```

- 주로 `while`문을 사용하여 클래스의 요소에 접근하는 방식을 사용


> `Set`의 경우 인덱스가 없는 관계로 일반 `for`문을 사용할 수 없기에 `for-each`문을 사용한다.  
> `for-each`문으로 자료 구조를 순회 중 수정/삭제에 대한 코드를 삽입시에 `Iterator`가 생성될 당시 컬렉션과 현재 내부 상태가 불일치하여 `ConcurrentModificationException`이 발생한다.

## ListIterator

```java
public interface ListIterator<E> extends Iterator<E> {
	boolean hasPrevious();
	E previous();
	int previousIndex();  
}
```

[List](Java-List.md) 전용 `Iterator` 인터페이스이다.  
`Iterator`는 단방향만 순회하는 반면에 `ListIterator`는 양방향으로의 이동이 가능하다.