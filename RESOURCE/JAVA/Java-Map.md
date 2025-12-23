
# Map

Map 인터페이스는 [컬렉션 프레임워크](Java-Collection-Framework.md)의 일부이며 키-값  페어를 저장하는데 사용한다.

- 검색, 삽입 및 삭제 작업에 유리
- 키는 고유하나 값은 중복 가능
- HashMap과 LinkedHashMap은 하나의 `null` 키를 허용하지만, TreeMap은 `null` 키를 허용하지 않음

```java
Map<Integer, String> map = new HashMap<>();  
map.put(1, "유어진");  
map.put(2, "유재석");  
map.put(3, "유지민");  
System.out.println(map); // {1=유어진, 2=유재석, 3=유지민}
```

##  계층 구조

구현 클래스는 다음과 같으며 주로 HashMap을 사용한다.

- [HashMap](Java-HashMap.md) : 빠른 접근, 삽입 및 삭제를 위해 해싱을 사용하여 키-값을 쌍을 저장한다다
- [LinkedHashMap](Java-LinkedHashMap.md) : 키의 삽입 순서를 유지
- TreeMap : 키-값 쌍을 자연 정렬 또는 사용자 지정 비교 연산자를 사용하여 정렬된 순서로 저장
- Hashtable : `null`를 허용하지 않는 동기화된 구현체

![Pasted image 20251223142118](../../GALLERY/Pasted%20image%2020251223142118.png)

## 메서드

### Map 기본 메서드
| 메서드                                        | 반환 타입     | 설명                                |
| ------------------------------------------ | --------- | --------------------------------- |
| `put(K key, V value)`                      | `V`       | 키에 값을 저장 (기존 값이 있으면 덮어쓰고 이전 값 반환) |
| `putAll(Map<? extends K, ? extends V> m)`  | `void`    | 다른 Map의 모든 엔트리를 복사                |
| `get(Object key)`                          | `V`       | 키에 대응하는 값 반환 (없으면 `null`)         |
| `getOrDefault(Object key, V defaultValue)` | `V`       | 키가 없을 경우 기본값 반환                   |
| `remove(Object key)`                       | `V`       | 키에 해당하는 엔트리 삭제 후 값 반환             |
| `remove(Object key, Object value)`         | `boolean` | 키와 값이 모두 일치할 때만 삭제                |
| `clear()`                                  | `void`    | 모든 엔트리 제거                         |
| `size()`                                   | `int`     | 저장된 엔트리 수 반환                      |
| `isEmpty()`                                | `boolean` | Map이 비어 있는지 여부                    |
| `containsKey(Object key)`                  | `boolean` | 해당 키 존재 여부                        |
| `containsValue(Object value)`              | `boolean` | 해당 값 존재 여부                        |
### 조회 뷰(View) 메서드
| 메서드          | 반환 타입                  | 설명                    |
| ------------ | ---------------------- | --------------------- |
| `keySet()`   | `Set<K>`               | 모든 키를 Set 형태로 반환      |
| `values()`   | `Collection<V>`        | 모든 값을 Collection으로 반환 |
| `entrySet()` | `Set<Map.Entry<K, V>>` | 키-값 쌍을 Set으로 반환       |
### Java 8 이후 추가된 메서드
| 메서드                                                                        | 반환 타입     | 설명                   |
| -------------------------------------------------------------------------- | --------- | -------------------- |
| `putIfAbsent(K key, V value)`                                              | `V`       | 키가 없을 때만 값 저장        |
| `compute(K key, BiFunction<? super K, ? super V, ? extends V> f)`          | `V`       | 키에 대해 새 값 계산         |
| `computeIfAbsent(K key, Function<? super K, ? extends V> f)`               | `V`       | 키가 없을 경우에만 값 계산 후 저장 |
| `computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> f)` | `V`       | 키가 있을 때만 값 재계산       |
| `merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> f)`   | `V`       | 기존 값과 병합             |
| `replace(K key, V value)`                                                  | `V`       | 키가 있을 때만 값 교체        |
| `replace(K key, V oldValue, V newValue)`                                   | `boolean` | 기존 값이 일치할 때만 교체      |
| `replaceAll(BiFunction<? super K, ? super V, ? extends V> f)`              | `void`    | 모든 엔트리 일괄 변경         |
| `forEach(BiConsumer<? super K, ? super V> action)`                         | `void`    | 모든 엔트리에 대해 반복 처리     |
### Map.Entry 주요 메서드
| 메서드                 | 반환 타입     | 설명        |
| ------------------- | --------- | --------- |
| `getKey()`          | `K`       | 엔트리의 키 반환 |
| `getValue()`        | `V`       | 엔트리의 값 반환 |
| `setValue(V value)` | `V`       | 값 변경      |
| `equals(Object o)`  | `boolean` | 엔트리 비교    |
| `hashCode()`        | `int`     | 해시코드 반환   |
```java
Set<Entry<Integer, String>> entries = map.entrySet();  
System.out.println(entries);
```

Entry에 대한 개념은 [여기](Entry.md)를 참고