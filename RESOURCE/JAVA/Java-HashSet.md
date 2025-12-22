# HashSet

- [Set](Java-Set.md) 인터페이스의 구현체
- 중복 요소를 허용하지 않음
- 내부적으로 [Hash Table](../CS/Map.md) 데이터 구조의 구현체인 HashMap을 사용
- `Serializable` 와 `Cloneable`를 구현함
- HashSet은 thread-safe 하지 않음

```java
Set<String> set = new HashSet<>();  
set.add("유어진");  
set.add("홍길동");  
set.add("유어진");  
  
System.out.println(set.size()); // 2  
System.out.println(set); // [유어진, 홍길동]
```

## 용량

HashSet의 용량은 Hash Table의 버킷 수를 나타낸다.  
기본 용량은 16이고 로드 팩터는 0.75이다.

> 여기서 Load Factor란 크기 조정을 위한 임계값이다.  
> 즉 Load Factor가 0.75라는 것은 전체 용량의 75%를 사용한다면 Hash Table를 Resizing한다.

## 생성자

```java
// 기본 용량(16)과 로드 팩터(0.75)를 갖는 새로운 빈 HashSet을 생성합니다.  
HashSet<String> set1 = new HashSet<>();  
  
// 지정된 초기 용량과 기본 로드 팩터(0.75)를 사용하여 빈 HashSet을 생성합니다.  
HashSet<String> set2 = new HashSet<>(36);  
  
// 지정된 초기 용량과 로드 팩터를 가진 빈 HashSet을 생성합니다.  
HashSet<String> set3 = new HashSet<>(36, 0.5F);  
  
// 지정된 컬렉션의 요소를 포함하는 새로운 HashSet을 생성합니다(중복은 자동으로 제거됨).  
 HashSet<String> set4 = new HashSet<>(set);
```

<br>

# 출처

[geeksforgeeks](https://www.geeksforgeeks.org/java/hashset-in-java/)  
