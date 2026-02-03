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