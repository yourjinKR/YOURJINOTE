# 정의
Functional Interface란  **Object Class의 메서드를 제외**하고 
'구현해야 할 추상 메서드가 하나만 정의된 인터페이스' 를 의미합니다.

> Object Class를 제외하는 이유는??
> 자바에서 사용되는 모든 객체들이 Object 객체를 상속하기에
> 인터페이스 구현체들이 Object 객체의 메서드를 굳이 재정의하지 않아도 그 메서드를 소유하기에

# 사용시 이점
1. 프로그램 검증이 쉽다
2. 계산한 값을 캐싱 하는 등의 최적화 가능
3. 스레드가 프로그램 상태를 공유하기 때문에 생기는 동시성 문제로부터 자유로운 편
4. 함수 자체를 재사용할 수 있음

# @FunctionalInterface
- 해당 인터페이스가 람다용임을 알려줌
- 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일이 되게 함
- 그 결과 유지 보수 과정에서 누가 실수로 메서드를 추가하지 못하게 막아주거나
  함수형이어야 하는 인터페이스가 다른 인터페이스를 상속한 경우 미리 확인할 수 있다.

```java
@FunctionalInterface  
public interface CustomFunctional {  
    void helloWorld();  
}
```

# Consumer Interface
- 역할 : 매개값만 받고 리턴값은 `void`
- 실행 메서드 : accept()
- 소비한다는 말은 사용만 할 뿐 리턴이 없다

| 인터페이스 명               | 추상 메서드                         | 설명                    |
| --------------------- | ------------------------------ | --------------------- |
| Consumer\<T>          | void accept(T t)               | 객체 T를 받아 소비           |
| BiConsumer\<T, U>     | void accept(T t, U u)          | 객체 T와 U를 받아 소비        |
| DoubleConsumer        | void accept(double value)      | double 값을 받아 소비       |
| IntConsumer           | void accept(int value)         | int 값을 받아 소비          |
| LongConsumer          | void accept(long value)        | long 값을 받아 소비         |
| ObjDoubleConsumer\<T> | void accept(T t, double value) | 객체 T와 double 값을 받아 소비 |
| ObjIntConsumer\<T>    | void accept(T t, int value)    | 객체 T와 int 값을 받아 소비    |
| ObjLongConsumer\<T>   | void accept(T t, long value)   | 객체 T와 long 값을 받아 소비   |

## Consumer
```java
// Consumer  
Consumer<String> sout = System.out::println;  
sout.accept("hello world");  
```
## BiConsumer
```java
// BiConsumer  
BiConsumer<String,Integer> intro = (name, age) -> System.out.println(name + " : " + age);  
intro.accept("유어진",26);
```
## 예시 : 3개 이상의 매개변수를 받는 람다함수
```java
interface TripleConsumer<T, U, K> {  
    void accept(T t, U u, K k);  
}  
  
public class TripleConsumerExample {  
    public static void main(String[] args) {  
        TripleConsumer<String, Double, Integer> c1 = (t, u, k) -> {  
            System.out.println(t);    
            System.out.println(u);    
            System.out.println(k);    
        };  
    }      
}
```

## 예시 : 에러가 발생할때마다 계속 실행
```java
public <T> void inputRepeat(Consumer<T> consumer, T params) {
    while (true) {
        try {
            consumer.accept(params);
            return;
        } catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
        }
    }
}
```


# Supplier Interface
- 역할 : 매개값이 없고 티런값만 반환
- 실행 메서드 : `getXXX()`

| 인터페이스 명         | 추상 메서드                 | 설명            |
| --------------- | ---------------------- | ------------- |
| Supplier\<T>    | T get()                | T 객체를 리턴      |
| BooleanSupplier | boolean getAsBoolean() | boolean 값을 리턴 |
| DoubleSupplier  | double getAsDouble()   | double 값을 리턴  |
| IntSupplier     | int getAsInt()         | int 값을 리턴     |
| LongSupplier    | long getAsLong()       | long 값을 리턴    |
## Supplier
```java
// Supplier  
Supplier<String> s1 = () -> new Object().toString();  
System.out.println(s1.get());  
```
## XXXSuplier
```java
// IntSupplier (BooleanSupplier, DoubleSupplier, LongSupplier들 또한 동일)  
IntSupplier s2 = () -> 100;  
System.out.println("백원 : " + s2);
```
# Function Interface
- 역할 : 매핑(타입 변환)하기
- 실행 메서드 : `applyXXX()`

| 인터페이스 명                   | 추상 메서드                         | 설명                    |
| ------------------------- | ------------------------------ | --------------------- |
| Function\<T, R>           | R apply(T t)                   | T를 받아 R을 리턴           |
| BiFunction\<T, U, R>      | R apply(T t, U u)              | T와 U를 받아 R을 리턴        |
| DoubleFunction\<R>        | R apply(double value)          | double 값을 받아 R을 리턴    |
| IntFunction\<R>           | R apply(int value)             | int 값을 받아 R을 리턴       |
| LongFunction\<R>          | R apply(long value)            | long 값을 받아 R을 리턴      |
| ToDoubleFunction\<T>      | double applyAsDouble(T value)  | T를 double로 변환하여 리턴    |
| ToIntFunction\<T>         | int applyAsInt(T value)        | T를 int로 변환하여 리턴       |
| ToLongFunction\<T>        | long applyAsLong(T value)      | T를 long으로 변환하여 리턴     |
| ToDoubleBiFunction\<T, U> | double applyAsDouble(T t, U u) | T와 U를 double로 변환하여 리턴 |
| ToIntBiFunction\<T, U>    | int applyAsInt(T t, U u)       | T와 U를 int로 변환하여 리턴    |
| ToLongBiFunction\<T, U>   | long applyAsLong(T t, U u)     | T와 U를 long으로 변환하여 리턴  |

## Function
```java
// int -> String  
Function<Integer, String> intToStr = String::valueOf;  
String str = intToStr.apply(123);
```
## BiFunction 
```java
// String, String -> int  
BiFunction<String, String, Integer> plusString = (t, u) -> {  
    return Integer.parseInt(t) + Integer.parseInt(u);  
};
```

# Operator Interface
- 역할 : 매개값 계산하여 동일한 타입으로 리턴
- 실행 메서드 : `applyXXX()`
- `Function`과 다른 점은 매개값을 리턴값으로 매핑하는 역할 보다는 매개값을 이용해서 연산을 수행한 후 동일한 타입으로 리턴 값을 제공하는 역할에 초점이 가 있다.

| 인터페이스 명              | 추상 메서드                                          | 설명                          |
| -------------------- | ----------------------------------------------- | --------------------------- |
| UnaryOperator\<T>    | T apply(T t)                                    | T를 받아 T를 리턴 (입출력 타입 동일)     |
| BinaryOperator\<T>   | T apply(T t1, T t2)                             | T 두 개를 받아 T를 리턴 (입출력 타입 동일) |
| IntUnaryOperator     | int applyAsInt(int operand)                     | int를 받아 int를 리턴             |
| LongUnaryOperator    | long applyAsLong(long operand)                  | long을 받아 long을 리턴           |
| DoubleUnaryOperator  | double applyAsDouble(double operand)            | double을 받아 double을 리턴       |
| IntBinaryOperator    | int applyAsInt(int left, int right)             | int 두 개를 받아 int를 리턴         |
| LongBinaryOperator   | long applyAsLong(long left, long right)         | long 두 개를 받아 long을 리턴       |
| DoubleBinaryOperator | double applyAsDouble(double left, double right) | double 두 개를 받아 double을 리턴   |

## 예시 : IntBinaryOperator를 활용한 배열 계산기
```java
class Operation {
    static int calculate(int[] arr, IntBinaryOperator o) {
        int result = arr[0];

        for (int i = 1; *i < arr.length; i++) {
            result = o.applyAsInt(result, arr[i]);
        }

        return result;
    }
}
```

아래와 같이 사용 가능
```java
public static void main(String[] args) {
    int[] numbers = {3, 1, 7, 6, 5};
    // 배열 요소의 모든 합 구하기
    int sum = Operation.calculate(numbers, (x, y) -> {
        return x + y;
    });
    System.out.println(sum);
    // 배열 요소의 모든 곱 구하기
    int mul = Operation.calculate(numbers, (x, y) -> {
        return x * y;
    });
    System.out.println(mul);
    // 배열 요소중 가장 큰 수 구하기
    int max = Operation.calculate(numbers, (x, y) -> *{
        int tmp;
        if(x > y)
            tmp = x;
        else
            tmp = y;
        return tmp;
    });
    System.out.println(max);
    // 배열 요소중 가장 작은 수 구하기
    int min = Operation.calculate(numbers, (x, y) -> {
        int tmp;*
        if(x < y)
            tmp = x;
        else
            tmp = y;
        return tmp;
    });
    System.out.println(min);
}
```

# Predicate
- 역할 : 매개값을 받고 true/false 리턴
- 실행 메서드 test()
- 매개값을 받아 참/거짓을 단정한다고 생각하면 된다.

| 인터페이스 명 | 추상 메서드 | 설명 |
|----------------|-------------|------|
| Predicate\<T> | boolean test(T t) | T를 받아 조건을 검사 (true/false 반환) |
| BiPredicate\<T, U> | boolean test(T t, U u) | T와 U를 받아 조건을 검사 |
| DoublePredicate | boolean test(double value) | double 값을 받아 조건을 검사 |
| IntPredicate | boolean test(int value) | int 값을 받아 조건을 검사 |
| LongPredicate | boolean test(long value) | long 값을 받아 조건을 검사 |
## 예시 : IntPredicate를 활용한 학생의 점수 판독기
```java
public static void main(String[] args) {

    List<Student> list = List.of(
            new Student("홍길동", 99),
            new Student("임꺽정", 76),
            new Student("고담덕", 36),
            new Student("김좌진", 77)
    );

    // int형 매개값을 받아 특정 점수 이상이면 true 아니면 false 를 반환하는 함수 정의
    IntPredicate scoring = (t) -> {
        return t >= 60;
    };

    for (Student student : list) {
        String name = student.name;
        int score = student.score;
		
        // 함수 실행하여 참 / 거짓 값 얻기
        boolean pass = scoring.test(score);
        
        if(pass) {
            System.out.println(name + "님 " + score + "점은 국어 합격입니다.");
        } else {
            System.out.println(name + "님 " + score + "점은 국어 불합격입니다.");
        }
    }
}
```

# Default Method
## Function 합성
함수 내부에 함수를 호출하는 것
```
f(g(x))
g(x)의 결과를 다시 f(x) 함수의 인자를 넣어준 것 처럼
```

```java
public static void main(String[] args) {

    Function<Integer, Integer> f = num -> (num - 4); // f(x)
    Function<Integer, Integer> g = num -> (num * 2); // g(x)

    // f(g(x))
    int a = f.andThen(g).apply(10);
    System.out.println(a); // (10 - 4) * 2 = 12

    // g(f(x)) - andThen을 반대로 해석하면 된다
    int b = f.compose(g).apply(10);
    System.out.println(b); // 10 * 2 - 4 = 16

}
```

# Collection Functional Interface
평소에 사용하던 것이 바로 이것임

| 컬렉션 / 스트림 메서드 | 사용되는 함수형 인터페이스 | 추상 메서드 | 설명 |
|------------------------|----------------------------|--------------|------|
| forEach() | Consumer\<T> | void accept(T t) | 요소를 하나씩 소비 (출력, 누적 등) |
| removeIf() | Predicate\<T> | boolean test(T t) | 조건을 만족하는 요소를 제거 |
| replaceAll() | UnaryOperator\<T> | T apply(T t) | 각 요소를 새 값으로 치환 |
| sort() | Comparator\<T> | int compare(T o1, T o2) | 두 요소를 비교하여 정렬 순서를 결정 |
| compute() / computeIfAbsent() / computeIfPresent() | BiFunction\<K, V, R> | R apply(K k, V v) | key와 value를 기반으로 새 value 계산 |
| merge() | BiFunction\<V, V, V> | V apply(V v1, V v2) | 두 value를 병합하여 새 value 생성 |
| forEach() (Map 전용) | BiConsumer\<K, V> | void accept(K k, V v) | key-value 쌍을 받아 처리 |
| replaceAll() (Map 전용) | BiFunction\<K, V, V> | V apply(K k, V v) | key-value를 받아 value 수정 |
| filter() | Predicate\<T> | boolean test(T t) | 조건에 맞는 요소만 필터링 |
| map() | Function\<T, R> | R apply(T t) | 요소를 다른 타입으로 변환 |
| flatMap() | Function\<T, Stream\<R>> | Stream\<R> apply(T t) | 중첩된 스트림을 평탄화 |
| distinct() | (내부적으로 equals/hashCode 사용) | - | 중복 제거 |
| sorted() | Comparator\<T> | int compare(T o1, T o2) | 요소 정렬 |
| peek() | Consumer\<T> | void accept(T t) | 중간 처리 (디버깅용) |
| reduce() | BinaryOperator\<T> | T apply(T t1, T t2) | 누적 연산 수행 |
| collect() | Collector\<T, A, R> | 다양한 조합 | 스트림을 최종 결과로 수집 |

# 출처
[자바 공식문서](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)
[](https://inpa.tistory.com/entry/☕-함수형-인터페이스-API#function_종류](https://inpa.tistory.com/entry/☕-함수형-인터페이스-API#function_종류](https://inpa.tistory.com/entry/☕-함수형-인터페이스-API#function_종류)