# TreeSet

- 중복 및 null 값을 허용하지 않음
- `NavigableSet` 인터페이스를 구현하며 탐색 메서드를 제공
	- `higher()`, `lower()`, `ceiling()`, `floor()`
- 동기화되어 있지 않으며 `Collection.synchronizedSet()`를 사용하여 동기화가 필요
- [이진 탐색 트리](../CS/BST-Binary-Search-Tree-이진-탐색-트리.md) 구조로 설계

## 탐색 메서드

### 1. higher(E e)

- 지정한 값 `e`보다 **큰 요소 중 가장 작은 값**을 반환한다.
- 존재하지 않으면 `null`을 반환한다.

### 2. lower(E e)

- 지정한 값 `e`보다 **작은 요소 중 가장 큰 값**을 반환한다.
- 존재하지 않으면 `null`을 반환한다.

### 3. ceiling(E e)

- 지정한 값 `e`와 **같은 값이 존재하면 해당 값**을 반환한다.
- 없다면 **e보다 큰 값 중 가장 작은 값**을 반환한다.

### 4. floor(E e)

- 지정한 값 `e`와 **같은 값이 존재하면 해당 값**을 반환한다.
- 없다면 **e보다 작은 값 중 가장 큰 값**을 반환한다.


```java
TreeSet<Integer> integers = new TreeSet<>();  
  
integers.add(7);  
integers.add(4);  
integers.add(9);  
integers.add(1);  
integers.add(5);  
  
System.out.println(integers.first()); // 1  
System.out.println(integers.last()); // 9  
  
System.out.println(integers.lower(4)); // 1  
System.out.println(integers.higher(7)); // 9  
  
System.out.println(integers.ceiling(4)); // 4  
System.out.println(integers.floor(7)); // 7  
  
  
integers.remove(9);  
  
System.out.println(integers); // [1, 4, 5, 7]
```

<br>

# 출처

[geeksforgeeks](https://www.geeksforgeeks.org/java/treeset-in-java-with-examples/)

