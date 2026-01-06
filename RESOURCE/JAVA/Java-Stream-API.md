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

해당 Stream을 생성시 여러 소스를 수정하지 않고도 

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

### `generate()`

```java
Stream<String> streamGenerated = Stream.generate(() -> "element").limit(10);
```

> `element`라는 문자열 10개를 연속으로 생성


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

### 지연 실행

연산을 즉시 실행하지 않고 최종 연산이 호출될 때까지 연산을 미룬다

```java
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
park은 세글자 이상인가?
네
세글자 이상이니 연산 실행
PARK
ko은 세글자 이상인가?
아니오
jo은 세글자 이상인가?
아니오
```

불필요한 연산을 최소화한다.

#### 단락 연산

`limit()`, `findFirst()`와 같은 경우는 중간 연산이지만 조건 요구시 스트림이 단락된다.  
즉, 이후의 요소들에 대해서는 연산이 실행되지 않으며 이는 Stream 성능 최적화의 핵심이다.

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

## 최종 연산

- Stream을 실행 (소비하는 개념에 가까움)
- 이 시점에서 모든 연산이 실행됨
- 호출된 스트림은 재사용 불가 (1회용)

### 대표 메서드

|메서드|결과|
|---|---|
|`forEach()`|소비|
|`toList()` / `collect()`|컬렉션|
|`count()`|long|
|`findFirst()`|Optional|
|`anyMatch()`|boolean|
|`reduce()`|누적 계산|

