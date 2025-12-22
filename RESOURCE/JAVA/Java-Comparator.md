
# Comparator

객체 정렬에 필요한 메서드를 선언

- Comparable : 기본 정렬 기준을 구현하는데 사용
- Comparator : 기본 정렬 기준 외에 다른 기준으로 정렬하고자 할 때 사용

```java
public int compareTo(Integer anotherInteger) {  
    return compare(this.value, anotherInteger.value);  
}
```

- this > another : 1
- this == another : 0
- this < another : -1

```java
Integer[] arr = { 20, 10, 50, 30, 40 };  
// Comparable
Arrays.sort(arr);  
System.out.println(Arrays.toString(arr)); // [10, 20, 30, 40, 50]  
  
// Comparator
Arrays.sort(arr, Collections.reverseOrder());  
System.out.println(Arrays.toString(arr)); // [50, 40, 30, 20, 10]
```