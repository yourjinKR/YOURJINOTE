해당 글은 gpt가 정리

## `Entry`란

`Entry`는 `Map` 컬렉션에서 **하나의 키-값 쌍(key-value pair)** 을 나타내는 **인터페이스**이다.  
정확한 이름은 `java.util.Map.Entry<K, V>`이며, `Map` 내부에 정의된 **중첩 인터페이스**다.  
이 인터페이스는 주로 `Map`의 `entrySet()` 메서드로 반환되는 엔트리들의 요소 타입으로 사용된다. ([Oracle Docs](https://docs.oracle.com/en/java/javase/24/docs/api/java.base/java/util/Map.Entry.html?utm_source=chatgpt.com "Map.Entry (Java SE 24 & JDK 24)"))

```java
interface Entry<K, V> {
	K getKey();
	V getValue();
	V setValue(V value);
	boolean equals(Object o);
	int hashCode();
	// 생략
}
```

공식 문서에 따르면, `Entry`는 키와 값으로 이루어진 하나의 항목을 표현하며, 특정 상황에서는 이 `Entry`를 통해 값(value)을 수정할 수도 있다.  
한편 `Map.entrySet()`의 반복(iteration)을 통해 얻은 `Entry` 객체는 **기저(underlying) Map과 연결된 경우**가 있고, 연결되지 않은 독립적인 경우도 있다. ([Oracle Docs](https://docs.oracle.com/en/java/javase/24/docs/api/java.base/java/util/Map.Entry.html?utm_source=chatgpt.com "Map.Entry (Java SE 24 & JDK 24)"))

### 기본 특징

- `Entry`는 `Map` 내부 요소 하나를 나타내는 **키-값 쌍**이다. ([Oracle Docs](https://docs.oracle.com/javase/8/docs/api/java/util/Map.Entry.html?utm_source=chatgpt.com "Map.Entry (Java Platform SE 8 )"))
    
- `getKey()` / `getValue()`로 각각 키와 값을 읽을 수 있다. ([Oracle Docs](https://docs.oracle.com/javase/8/docs/api/java/util/Map.Entry.html?utm_source=chatgpt.com "Map.Entry (Java Platform SE 8 )"))
    
- `setValue(V value)`를 구현하는 경우 값(value)을 변경할 수 있다. ([Oracle Docs](https://docs.oracle.com/javase/8/docs/api/java/util/Map.Entry.html?utm_source=chatgpt.com "Map.Entry (Java Platform SE 8 )"))
    
- `equals()` / `hashCode()`는 **키와 값이 모두 같을 때** 같은 엔트리로 판단한다. ([Oracle Docs](https://docs.oracle.com/javase/8/docs/api/java/util/Map.Entry.html?utm_source=chatgpt.com "Map.Entry (Java Platform SE 8 )"))
    

공식 정의 문서는 Oracle Java SE API 문서이다.

- Oracle Java SE API: `Map.Entry` 공식 문서  
    [https://docs.oracle.com/en/java/javase/24/docs/api/java.base/java/util/Map.Entry.html](https://docs.oracle.com/en/java/javase/24/docs/api/java.base/java/util/Map.Entry.html) ([Oracle Docs](https://docs.oracle.com/en/java/javase/24/docs/api/java.base/java/util/Map.Entry.html?utm_source=chatgpt.com "Map.Entry (Java SE 24 & JDK 24)"))
    

## 주로 사용되는 경우

보통 `Map`의 전체 요소를 순회(iterate)할 때 `entrySet()`과 함께 사용한다.  
예를 들어 `HashMap`에 저장된 모든 키와 값을 하나씩 읽을 때 `Entry`를 활용한다.

## 메서드 설명

- `K getKey()`  
    키(key)를 반환한다. ([Oracle Docs](https://docs.oracle.com/javase/8/docs/api/java/util/Map.Entry.html?utm_source=chatgpt.com "Map.Entry (Java Platform SE 8 )"))
    
- `V getValue()`  
    값(value)을 반환한다. ([Oracle Docs](https://docs.oracle.com/javase/8/docs/api/java/util/Map.Entry.html?utm_source=chatgpt.com "Map.Entry (Java Platform SE 8 )"))
    
- `V setValue(V value)`  
    값을 새 값으로 바꾼다. 이때 `Map`이 수정 가능하면 Map까지 변경이 반영된다. ([Oracle Docs](https://docs.oracle.com/javase/8/docs/api/java/util/Map.Entry.html?utm_source=chatgpt.com "Map.Entry (Java Platform SE 8 )"))
    

## 예시 코드

### `Map.Entry`를 이용해서 Map 순회하기

```java
import java.util.HashMap;
import java.util.Map;

public class EntryExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);

        // entrySet()을 이용한 순회
        for (Map.Entry<String, Integer> entry : map.entrySet()) {
            String key = entry.getKey();
            Integer value = entry.getValue();
            System.out.println(key + " => " + value);
        }
    }
}
```

### `Entry`의 값 수정 예시

```java
import java.util.HashMap;
import java.util.Map;

public class EntryModifyExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        map.put("score", 10);

        for (Map.Entry<String, Integer> entry : map.entrySet()) {
            if (entry.getKey().equals("score")) {
                entry.setValue(entry.getValue() + 5);
            }
        }

        System.out.println(map.get("score")); // 출력: 15
    }
}
```
