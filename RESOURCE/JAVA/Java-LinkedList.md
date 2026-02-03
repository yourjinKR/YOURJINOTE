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