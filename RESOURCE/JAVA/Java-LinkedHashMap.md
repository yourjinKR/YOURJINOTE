# LinkedHashMap

- `HashMap`을 상속하고 [Map 인터페이스](Java-Map.md)를 구현
- 고유한 키-값 페어를 저장
- 삽입 순서 유지
- 하나의 `null` 키와 여러 개의 `null` 값을 허용
- thread-safe를 위해서는 `Collections.synchronizedMap()`를 사용

```java
public class LinkedHashMap<K,V>  
    extends HashMap<K,V>  
    implements Map<K,V>  {    
}
```

`LinkedHashMap`은 `HashMap`을 상속하고 `Map` 인터페이스를 구현함.

## 내부 동작 방식

데이터를 이중 연결 리스트와 유사한 노트 형태로 저장하며 다음과 같이 데이터를 관리

```java
public class LinkedHashMap<K,V> {

	static class Entry<K,V> extends HashMap.Node<K,V> {  
	    Entry<K,V> before, after;  // 앞 뒤를 알고 있음
	    Entry(int hash, K key, V value, Node<K,V> next) {  
	        super(hash, key, value, next);  
	    }  
	}
	
}
```

- `key` : HashMap을 상속하므로 데이터는 키-값 쌍 형태로 저장
- `value` : 모든 `key`에는 해당 `key`와 연결된 값이 있으며 해당 매개변수는 키의 값을 저장함
- `after` : 삽입 순서를 저장하므로, 이 값에서는 `LinkedHashMap`의 다음 노드 주소를 저장
- `before` : 이전 노드 주소를 저장

## 생성자

```java
// 기본 초기 용량(16) 및 로드 팩터(0.75)를 사용하여 빈 LinkedHashMap을 생성합니다.
LinkedHashMap<K, V> lhm = new LinkedHashMap<>();
// 지정된 용량과 로드 팩터로 LinkedHashMap을 생성합니다.
LinkedHashMap<K, V> lhm = new LinkedHashMap<>(20, 0.75f);
// 지정된 맵의 모든 요소를 ​​포함하는 LinkedHashMap을 생성하며, 요소 삽입 순서를 유지합니다.
LinkedHashMap<K, V> lhm = new LinkedHashMap<>(existingMap);

```

## 메서드

`LinkedHashMap`에서 주목할 점은 특정 요소 변경시 순서에 맞게 대입한다는 것이다.

```java
LinkedHashMap<Integer, String> map = new LinkedHashMap<>();  
  
// 값 넣기  
map.put(2, "유재석");  
map.put(1, "유어진");  
map.put(3, "유지민");  
System.out.println(map); // {2=유재석, 1=유어진, 3=유지민}  
  
map.put(1, "기존 순서에 맞게 값 변경");  
System.out.println(map);  
// {2=유재석, 1=기존 순서에 맞게 값 변경, 3=유지민}
```