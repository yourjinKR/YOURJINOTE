# Stream API

데이터의 흐름을 표현하고 그 흐름에 함수형 연산을 적용할 수  있는 API

```java
public interface Stream<T> extends BaseStream<T, Stream<T>> {
	void forEach(Consumer<? super T> action);
}
```

스트림은 아래와 같은 로직을 거친다.

- 스트림 생성
- 중간 연산
- 최종 연산

## 기존 for문과의 차이점

### 내부 반복

기존의 자바 컬렉션은 개발자가 **직접 반복문을 돌리는 형식**이다.

```java
for (String name : names) {  
    if (name.length() == 3) {  
        System.out.println(name);  
    }  
}
```

> 기존 방식은 반복문, 조건문, 인덱스 관리 등 절차적 로직이 섞여 있다.  
> 병렬 처리에 불리하다.

반면 Stream을 이용시 로직을 흘러가듯이(?) 선언하고 사용 가능하다.

```java
names.stream()  
        .filter(name -> name.length() == 3)  
        .forEach(System.out::println);
```

> 직접 `for`문을 통해 반복하는 것이 아닌 `Stream` 내부에서 처리하며 이를 **내부 반복**이라고 한다.

## 스트림 주요 특징

- 데이터 원본 불변성
- 일회성
- 내부 반복
- 파이프라인 구성
- 지연 연산

### 지연 연산

```java
// 길이가 3 이상인 글자를 대문자로 변경
List<String> list1 = names.stream()  
        .filter(name -> name.length() >= 3)  
        .map(name -> name.toUpperCase(Locale.ROOT))  
        .toList();
```

- `stream()`을 실행하여 컬렉션을 스트림으로 변환 (아무런 연산이 실행되지 않음)
- `filter()`, `map()`은 중간 연산이며 이 또한 실행되지 않음
- `toList()`는 최종 연산에 해당되며 해당 시점에서 스트림이 실행된다.

# 파이프라인 구성

- 스트림 생성
- 중간 연산
- 최종 연산

> 스트림 코드는 항상 해당 구조를 벗어나지 않는다.

## 스트림 생성

> `stream()`은 스트림 파이프 라인을 구성하는 것이고, 실제 연산은 최종 연산 시점에서 실행됩니다.

### 빈 스트림

```java
public static<T> Stream<T> empty() {  
    return StreamSupport.stream(Spliterators.<T>emptySpliterator(), false);  
}
```

```java
Stream<String> emptyString = Stream.empty();  
Stream<Integer> emptyInteger = Stream.empty();
```

### 컬렉션

```java
Stream<String> stringStream = List.of("dd","aa","cc").stream();
```

> Collection은 데이터 저장을 위한 목적이고, 스트림은 데이터를 흘려보내며 가공하는 역할이다.

### 배열

```java
String[] arr = {"a", "b", "c"};  
Stream<String> streamOfArrayFull = Arrays.stream(arr);  
Stream<String> streamOfArrayPart = Arrays.stream(arr, 1, 3);
```

### 기본형 스트림

```java
IntStream range = IntStream.range(1, 5); // 1~4  
IntStream intStream = IntStream.rangeClosed(1, 5);// 1~5
```

- 기본형 타입에 대해서는 별도의 `Stream` 타입을 제공하며 성능면에서 효율적이다.
	- 내부 요소: **primitive**
	- 박싱하지 않음
	- 연산 성능 증가
	- 메모리 효율 증가

```java
int sum = IntStream.range(1, 10).sum();
double avg = IntStream.range(1, 10).average().orElse(0);
int max = IntStream.range(1, 10).max().orElse(0);
```

- 직관적인 코드를 작성 할 수 있다.

#### 기본형 스트림과 비교

기본형 데이터를 Stream으로 다룬다면 다음과 같은 특징을 가진다.

```java
// 일반적인 스트림 방식
Stream<Integer>
```

- 내부 요소: **Wrapping Class**
    
- 연산 시:
    - 오토 박싱 / 언박싱 발생
    - **힙 메모리 사용**
    - GC 부담 증가

#### 다시 객체 스트림으로 변환

```java
Stream<Integer> boxed =
    IntStream.range(1, 5).boxed();
```

단, 해당 연산은 다시 객체를 생성하기에 그 만큼의 비용이 발생한다!

### 파일 스트림

```java
String filePath = "example.txt";  
String content = "Hello, Java File Stream!";  
  
// 1. 파일 출력 (파일에 쓰기)  
try (FileOutputStream fos = new FileOutputStream(filePath)) {  
    byte[] bytes = content.getBytes();  
    fos.write(bytes);  
    System.out.println("파일 씀");  
} catch (IOException e) {  
    e.printStackTrace();  
}  
  
// 2. 파일 입력 (파일에서 읽기)  
try (FileInputStream fis = new FileInputStream(filePath)) {  
    int data;  
    System.out.print("파일 내용 읽기: ");  
    while ((data = fis.read()) != -1) { // -1은 파일의 끝(EOF)을 의미  
        System.out.print((char) data); // 읽어온 바이트를 문자로 변환하여 출력  
    }  
    System.out.println();  
} catch (IOException e) {  
    e.printStackTrace();  
}
```

### 무한 스트림

`limit()`과 같은 단락 연산을 사용하지 않는다면 무한 스트림이기에 인스턴스를 무한대로 생성한다.

```java
// generate  
Stream<Double> randoms = Stream.generate(Math::random);  
// iterator 
Stream<Integer> nums = Stream.iterate(1, n -> n + 1);  
// 사용  
Stream.iterate(1, n -> n + 1)  
        .limit(5)
        .forEach(System.out::println);  
  
// 반드시 limit 설정  
randoms.limit(5).forEach(System.out::println);
```

#### `generate()`

```java
Stream<String> streamGenerated = Stream.generate(() -> "element").limit(10);
```

> `element`라는 문자열 10개를 연속으로 생성

### `builder()`

빌더 사용시 원하는 타입을 추가로 지정해야 한다

```java
public static<T> Builder<T> builder() {  
    return new Streams.StreamBuilderImpl<>();  
}
```

```java
Stream<String> streamBuilder =  
        Stream.<String>builder()  
                .add("a")  
                .add("b")  
                .add("c")  
                .build();
```


## 중간 연산

- Stream을 반환 -> 체이닝 가능
- 지연 실행
- 여러개 연결 가능

### 대표 메서드

| 메서드                  | 역할     |
| -------------------- | ------ |
| `filter()`           | 조건 필터링 |
| `map()`              | 변환     |
| `sorted()`           | 정렬     |
| `distinct()`         | 중복 제거  |
| `limit()` / `skip()` | 개수 제한  |
| `flatMap()`          | 평탄화    |
| `peek()`             | 디버깅 전용 |

#### 단락 연산

```java
.limit(10) // 앞에서 10개
.skip(5) // 앞에서 5개 건너뜀
```

`limit()`, `findFirst()`와 같은 경우는 중간 연산이지만 **조건 요구시 스트림이 단락**된다.  
즉, 이후의 요소들에 대해서는 연산이 실행되지 않으며 이는 **Stream 성능 최적화의 핵심**이다.

#### `sorted()`

정렬은 **모든 요소를 알아야 실행 가능**하다.
그러므로 완전한 지연 연산이 아닌 **내부적으로 전체 요소를 한번 모은다.**

그렇기에 `filter()`를 실행해야 한다면 먼저 `filter`를 실행시켜 주는 것이 효율적이다.

```java
nums.stream()
    .filter(n -> n > 3) // 필터링을 먼저 하여 정렬할 요소를 줄인다.
    .sorted()
    .forEach(System.out::println);
```

```java
class Member implements Comparable<Member> {

	int age;

    @Override
    public int compareTo(Member o) {
        return this.age - o.age; // 나이 오름차순
    }
}
```

해당 연산을 수행하기 위해서는 Stream의 요소가 `Comparable`를 구현해야 한다.  

혹은 [Comparator](Java-객체비교.md#Comparator)를 사용하여 연산시 정렬 조건을 지정할 수 있다.

```java
List<Member> list3 = members.stream()  
        .sorted(Comparator.comparing(Member::getAge).reversed())  
        .toList();
```

> 각 연산마다 정렬에 대한 기준이 달라질 수 있으므로 `Comparator`를 사용하도록 권장한다.

#### `distinct()`

```java
stream.distinct();
```

- 내부적으로 `Set`
- `equals` / `hashCode` 기반으로 같은 객체인지 검증한다.

### `flatMap()`

```java
List<Item> items = List.of(  
        new Item(1),  
        new Item(2),  
        new Item(3),  
        new Item(4),  
        new Item(5),  
        new Item(6)  
);  
  
Data data = new Data(1, items);  
  
Stream.of(data)  
        .flatMap(d -> d.getItems().stream())  
        .forEach(i -> System.out.println(i.getId()));
```

> **map과 flatMap의 차이**  
> map은 요소를 1:1로 변환하여 중첩 구조를 유지함  
> 반면 flatMap은 요소를 여러 개로 펼쳐 하나의 스트림으로 평탄화 한다.  
> 주로 컬렉션이나 Optional 같은 중첩 구조를 처리할 때 사용한다.

### 지연 실행

연산을 즉시 실행하지 않고 최종 연산이 호출될 때까지 연산을 미룬다

```java
List<String> names = List.of("yoo", "ha", "kim");

names.stream()
     .filter(n -> {
         System.out.println("filter: " + n);
         return n.length() <= 3;
     })
     .map(n -> {
         System.out.println("map: " + n);
         return n.toUpperCase();
     })
     .forEach(System.out::println);
```

스트림은 요소 단위로 하나씩 처리되며 다음과 같이 실행된다.

```
yoo은 세글자 이상인가?
네
세글자 이상이니 연산 실행
YOO
ha은 세글자 이상인가?
아니오
kim은 세글자 이상인가?
네
세글자 이상이니 연산 실행
KIM
```

불필요한 연산을 최소화한다.

## 최종 연산

- Stream을 실행 (소비하는 개념에 가까움)
- 이 시점에서 모든 연산이 실행됨
- 호출된 스트림은 재사용 불가 (1회용)

### 대표 메서드

| 메서드                      | 결과       |
| ------------------------ | -------- |
| `forEach()`              | 소비       |
| `toList()` / `collect()` | 컬렉션      |
| `count()`                | long     |
| `findFirst()`            | Optional |
| `anyMatch()`             | boolean  |
| `reduce()`               | 누적 계산    |

#### `forEach()`

```java
stream.forEach(System.out::println);
```

출력, 로그, 외부 시스템 호출 등에서 사용함

#### `toList()` / `collect()`

```java
List<String> result = stream.toList(); // Java 16

List<String> result =
    stream.collect(Collectors.toList());
```

```java
// Map
Map<String, Integer> map =
    stream.collect(Collectors.toMap(
        Member::getName,
        Member::getAge
    ));
```

### `count()`

```java
long cnt = stream.count();
```

#### `findFirst()` / `findAny()`

두 메서드는 요소 하나를 가져오는 것이며 각각의 특징을 고려하여 사용

```java
// 순서 보장
Optional<T> result = stream.findFirst();

// 병렬 스트림에서 빠름
Optional<T> result = stream.findAny();
```

#### `anyMatch()` /  `allMatch()` / `noneMatch()`

조건 검사에 사용함

| 메서드       | 의미      |
| --------- | ------- |
| anyMatch  | 하나라도 만족 |
| allMatch  | 모두 만족   |
| noneMatch | 모두 불만족  |

```java
// 해당 그룹에 어른이 있는가
boolean hasAdult =
    members.stream().anyMatch(m -> m.getAge() >= 20);
```

#### `reduce()`

```java
// nums의 합 구하기
int result1 = Arrays.stream(nums)  
        .reduce(0, Integer::sum);  
  
int result2 = Arrays.stream(nums)  
        .sum();  
```

- 모든 최종 연산의 근본
- 가독성 측면에서는 좋지 않음
- **collect가 가능한 경우 reduce 지양**

### collector()

스트림의 요소들을 **원하는 자료구조나 결과 형태로 수집하는 최종 연산**

#### list와 Set의 간단한 버전

```java
List<String> list =
    stream.collect(Collectors.toList());
    
Set<String> set =
    stream.collect(Collectors.toSet());
        
```

#### 직접 자료구조 지정

```java
LinkedList<String> list =
    stream.collect(Collectors.toCollection(LinkedList::new));
```

#### toMap()

```java
List<String> example = List.of("apple", "banana", "kiwi", "lemon");  
Map<Integer, String> collect = example.stream().collect(Collectors.toMap(  
        String::length,  // 키
        Function.identity(),  // 값
        (oldValue, newValue) -> oldValue, // 키 충돌시 해결 방법,  
        LinkedHashMap::new // Map 타입 이외에 특정 컬렉션으로 저장  
));
```

#### 불변 컬렉션과 toList()

```java
List<Integer> list = members.stream()  
        .map(member -> member.age + 1)  
        .toList();  
  
List<Integer> list2 = members.stream()  
        .map(member -> member.age + 2)  
        .collect(Collectors.toUnmodifiableList());  
  
// UnsupportedOperationException  
// list.add(100);  
// list2.add(100);
```

`toList()`와 `Collectors.toUnmodifiableList()`를 사용하면 불변 컬렉션으로 데이터를 다룰 수 있다.

####  문자열 수집

```java
List<Person> people = List.of(  
        new Person("어진1", 26),  
        new Person("어진2", 27),  
        new Person("어진3", 28)  
);  
  
String joining3 = people.stream()  
        .map(Person::name)  
        .collect(Collectors.joining(", "));
        
System.out.println(joining3);
// 어진1, 어진2, 어진3
```

#### 통계 컬렉션

```java
Double average = people.stream()  
        .collect(Collectors.averagingInt(Person::age));  
System.out.println(average);
```

- 평균 : `averagingInt` / `averagingLong` / `averagingDouble`
- 합계 : `summingInt` / `summingLong` / `summingDouble`

```java
Optional<Member> oldest =
    members.stream()
           .collect(Collectors.maxBy(
               Comparator.comparingInt(Member::getAge)
           ));
```

- 최대 / 최소 : `maxBy` / `minBy`

```java
// 통계 요약  
IntSummaryStatistics summaryStatistics = people.stream()  
        .collect(Collectors.summarizingInt(Person::age));  
        
System.out.println(summaryStatistics.getAverage());  
System.out.println(summaryStatistics.getCount());  
System.out.println(summaryStatistics.getMax());  
System.out.println(summaryStatistics.getMin());  
System.out.println(summaryStatistics.getSum());
```

`summarizingXXXX()`를 통해 한번의 연산으로 모든 통계 수치를 얻을 수 있다.

#### 다운 스트림

그룹핑된 각 그룹에 대해 또 한 번 collect를 수행하는 구조  
자세한 내용은 [여기](Java-Down-Stream.md)를 참고

<br>

# 출처

[남궁성 - 자바의 정석](https://www.youtube.com/watch?v=AOw4cCVUJC4&list=PLW2UjW795-f6xWA2_MUhEVgPauhGl3xIp&index=164)  
[인프런 - 김영한의 실전 자바](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EA%B3%A0%EA%B8%89-3)  
[티스토리 - Collection.toMap 사용법](https://ttl-blog.tistory.com/1232)  
