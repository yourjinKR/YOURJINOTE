
# java.util.Collections 클래스 메서드 정리

`java.util.Collections`는 [컬렉션 프레임워크](Java-Collection-Framework.md)를 보조하기 위한 유틸리티 클래스다.  
모든 메서드는 static이며, 컬렉션 정렬, 검색, 변환, 불변 처리, 동기화 등을 담당한다.  
실무에서는 List, Set, Map을 직접 상속한 구현체보다 이 클래스의 메서드를 활용하는 경우가 매우 많다.

- Collections는 컬렉션을 직접 구현하지 않고 기능을 보조하는 유틸리티 클래스다
- 모든 메서드는 `static`이며 인스턴스 생성이 불가능하다
- 정렬, 불변 처리, 동기화, 검색 등 실무 핵심 기능이 집중되어 있다
- Java 9 이후에는 `List.of` 등과 병행하여 사용된다

## 정렬 관련 메서드

### sort

- `static <T extends Comparable<? super T>> void sort(List<T> list)`
- `static <T> void sort(List<T> list, Comparator<? super T> c)`

리스트를 오름차순 또는 사용자 정의 기준으로 정렬한다.

```java
List<Integer> numbers = new ArrayList<>(List.of(3, 1, 2));
Collections.sort(numbers);
// [1, 2, 3]

Collections.sort(numbers, Comparator.reverseOrder());
// [3, 2, 1]
````

### reverse

- `static void reverse(List<?> list)`
    

리스트의 순서를 반대로 뒤집는다.

```java
Collections.reverse(numbers);
// [1, 2, 3]
```

### shuffle

- `static void shuffle(List<?> list)`
    

리스트 요소를 무작위로 섞는다.

```java
Collections.shuffle(numbers);
```

### swap

- `static void swap(List<?> list, int i, int j)`
    

두 인덱스의 요소를 교환한다.

```java
Collections.swap(numbers, 0, 2);
```

## 검색 및 비교 메서드

### binarySearch

- `static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key)`
    
- `static <T> int binarySearch(List<? extends T> list, T key, Comparator<? super T> c)`
    

정렬된 리스트에서 이진 탐색을 수행한다.  
정렬되지 않은 리스트에서 사용하면 결과를 보장하지 않는다.

```java
List<Integer> sorted = List.of(1, 2, 3, 4, 5);
int index = Collections.binarySearch(sorted, 3);
// index = 2
```

### min / max

- `static <T extends Object & Comparable<? super T>> T min(Collection<? extends T> coll)`
    
- `static <T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll)`
    

컬렉션 내 최소값, 최대값을 반환한다.

```java
int min = Collections.min(sorted);
int max = Collections.max(sorted);
```

#### 요소가 객체일 때

컬렉션의 요소 타입이 커스텀 클래스인 경우,  
`Comparable`을 구현하여 객체 간의 자연 정렬 기준을 정의할 수 있다.

```java
Person maxPerson = Collections.max(people);  
System.out.println(maxPerson); // Person{age=40}
```

```java
class Person implements Comparable<Person> {  
    int age;  
  
    public Person(int age) {  
        this.age = age;  
    }  
  
    @Override  
    public int compareTo(Person o) {  
        // return this.age > o.age ? 1 : this.age == o.age ? 0 : -1;  
        return Integer.compare(this.age, o.age);  
    }  
  
    @Override  
    public String toString() {  
        return "Person{" +  
                "age=" + age +  
                '}';  
    }  
}
```


## 불변 컬렉션 생성 메서드

이 메서드들은 원본 컬렉션을 감싸는 읽기 전용 뷰를 반환한다.  
구조 변경 시 `UnsupportedOperationException`이 발생한다.

### unmodifiableList / Set / Map

- `unmodifiableList(List<? extends T> list)`
    
- `unmodifiableSet(Set<? extends T> set)`
    
- `unmodifiableMap(Map<? extends K, ? extends V> map)`
    

```java
List<String> mutable = new ArrayList<>();
mutable.add("A");

List<String> readonly = Collections.unmodifiableList(mutable);
// readonly.add("B"); // 예외 발생
```

원본 컬렉션이 변경되면 불변 뷰에도 반영된다.

## 동기화 래퍼 메서드

멀티스레드 환경에서 컬렉션을 안전하게 사용하기 위한 synchronized 래퍼를 제공한다.

### synchronizedList / Set / Map

- `synchronizedList(List<T> list)`
    
- `synchronizedSet(Set<T> set)`
    
- `synchronizedMap(Map<K, V> map)`
    

```java
List<String> syncList =
    Collections.synchronizedList(new ArrayList<>());

synchronized (syncList) {
    for (String s : syncList) {
        System.out.println(s);
    }
}
```

반복(iteration) 시에는 반드시 외부에서 synchronized 블록을 사용해야 한다.

## 채우기 및 대체 메서드

### fill

- `static <T> void fill(List<? super T> list, T obj)`
    

리스트의 모든 요소를 동일한 값으로 채운다.

```java
List<String> list = new ArrayList<>(List.of("A", "B", "C"));
Collections.fill(list, "X");
// [X, X, X]
```

### replaceAll

- `static <T> boolean replaceAll(List<T> list, T oldVal, T newVal)`
    

특정 값들을 다른 값으로 치환한다.

```java
Collections.replaceAll(list, "X", "Y");
```

## 빈 컬렉션 제공 메서드

불변(empty) 컬렉션을 반환하며, null 대신 사용하기 좋다.

- `emptyList()`
    
- `emptySet()`
    
- `emptyMap()`
    

```java
List<String> empty = Collections.emptyList();
```

## 단일 요소 컬렉션 생성

- `singleton(T o)`
    
- `singletonList(T o)`
    
- `singletonMap(K key, V value)`
    

```java
List<String> single = Collections.singletonList("A");
```

## 빈도 및 포함 여부 확인

### frequency

- `static int frequency(Collection<?> c, Object o)`
    

특정 요소의 등장 횟수를 반환한다.

```java
List<String> list = List.of("A", "B", "A");
int count = Collections.frequency(list, "A");
// 2
```

### disjoint

- `static boolean disjoint(Collection<?> c1, Collection<?> c2)`


두 컬렉션이 공통 요소를 가지지 않으면 true를 반환한다.

```java
boolean result =
    Collections.disjoint(List.of(1, 2), List.of(3, 4));
```

# 출처

[Oracle](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Collections.html)