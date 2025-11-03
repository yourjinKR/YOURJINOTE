#java #collection #list #평탄화 #flatMap

2차원 리스트를 1차원 리스트로 변경할때 사용

```java
List<List<Integer>> matrix = new ArrayList<>();  
matrix.add(List.of(1, 2, 3));  
matrix.add(List.of(4, 5, 6));  
matrix.add(List.of(7, 8, 9));  
  
List<Integer> flattenedList = matrix.stream()  [^1]
        .flatMap(List::stream)  
        .toList();  
  
System.out.println("Flattened List: " + flattenedList);
```
