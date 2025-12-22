# LinkedHashSet

- [Set](Java-Set.md) 인터페이스를 구현함
- 중복을 허용하지 않으며 `null` 값을 하나만 허용
- [HashSet](Java-HashSet.md)과 LinkedList의 기능을 결합하여 요소의 삽입 순서를 유지한다
- Set, Cloneable, Serialized

```java
Set<String> hashSet = Set.of("1번", "2번", "3번");  
LinkedHashSet<String> linkedHashSet = new LinkedHashSet<>();  
  
linkedHashSet.add("1번");  
linkedHashSet.add("2번");  
linkedHashSet.add("3번");  
  
System.out.println(hashSet); // [1번, 3번, 2번]  
System.out.println(linkedHashSet); // [1번, 2번, 3번]
```

해당 컬렉션 또한 HashSet과 같은 생성자를 가지고 있다.

<br>

# 출처

[geeksforgeeks](https://www.geeksforgeeks.org/java/linkedhashset-in-java-with-examples/)  