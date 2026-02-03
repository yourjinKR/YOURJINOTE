# HashMap

- `AbstractMap`을 상속하고 [Map 인터페이스](Java-Map.md)를 구현함
- 내부적으로 [해싱](../CS/Hash-Function-해시-함수.md)을 사용하기에 상수 시간의 키 기반 검색, 삽입, 제거가 가능
- 동기화시 `Collections.synchronizedMap()`을 사용
- 삽입 순서가 유지되지 않음

```java
public class HashMap<K,V> 
	extends AbstractMap<K,V>  
	implements Map<K,V>, Cloneable, Serializable {}
```

```java
Map<Integer, String> map = new HashMap<>();  
map.put(1, "유어진");  
map.put(2, "유재석");  
map.put(3, "유지민");  
System.out.println(map); // {1=유어진, 2=유재석, 3=유지민}
```

## 용량

HashMap의 용량은 항목을 저장하기 위해 사용할 수 있는 버킷의 개수이다.

- 기본 용량 : 16
- 기본 LoadFactor : 0.75

## 생성자

```java
// 이는 초기 용량 16, 로드 팩터 0.75로 HashMap 인스턴스를 생성하는 기본 생성자입니다.
HashMap<K, V> hm = new HashMap<K, V>();

// 지정된 초기 용량과 0.75의 로드 팩터를 가진 HashMap 인스턴스를 생성합니다.
HashMap<K, V> hm = new HashMap<K, V>(int initialCapacity);

// 지정된 초기 용량과 지정된 로드 팩터를 사용하여 HashMap 인스턴스를 생성합니다.
HashMap<K, V> hm = new HashMap<K, V>(int initialCapacity, float loadFactor);

// 지정된 맵과 동일한 매핑을 가진 HashMap 인스턴스를 생성합니다.
HashMap<K, V> hm = new HashMap<K, V>(Map map);
```

## 사용 예시

```java
Map<Integer, String> map = new HashMap<>();  
  
// 값 넣기  
map.put(1, "유어진");  
map.put(2, "유재석");  
map.put(3, "유지민");  
System.out.println(map); // {1=유어진, 2=유재석, 3=유지민}  
  
// 여러 값 넣기  
Map<Integer, String> map2 = Map.of(4, "유어진", 5, "유재석");  
map.putAll(map2);  
System.out.println(map);  
  
// 값 제거  
map.remove(4);  
map.remove(5, "유재석");  
System.out.println(map);  
  
// 값 얻기  
System.out.println(map.get(1));  
System.out.println(map.getOrDefault(5, "없음 말고"));  
  
// 키, 값 여부 확인  
System.out.println(map.containsKey(5));  
System.out.println(map.containsValue("유지민"));  
  
// 키와 값을 얻기  
Set<Integer> keys = map.keySet();  
System.out.println(keys);  
  
Collection<String> values = map.values();  
System.out.println(values);
```

# 출처

[geeksforgeeks](https://www.geeksforgeeks.org/java/java-util-hashmap-in-java-with-examples/)  
