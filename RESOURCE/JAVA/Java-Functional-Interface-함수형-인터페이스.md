# 함수형 인터페이스

```java
@FunctionalInterface  
interface Myfunction {  
    int max(int a, int b);  
}
```

- 추상 메서드가 하나 뿐인 인터페이스를 의미
- `@FunctionalInterface` 어노테이션을 통해 함수형 인터페이스임을 컴파일러에게 알려줄 수 있다.

> 어노테이션이 붙은 상태로 2개 이상의 추상 메서드를 선언시 컴파일 에러 발생

## 구현 방식

```java
@FunctionalInterface  
interface Myfunction {  
    int max(int a, int b);  
}

Myfunction oldFunction = new Myfunction() {  
    @Override  
    public int max(int a, int b) {  
        return a > b ? a : b;  
    }  
};
```

`oldFunction`은 `MyFunction` 인터페이스를 구현한 익명 클래스의 객체 생성 방식이다.

```java
Myfunction myfunction = (a, b) -> a > b ? a: b;
```

람다식을 통해 위와 같이 간단히 구현할 수 있다.

> [람다식](Java-Lambda-람다.md)을 다루기 위한 참조변수의 타입은 함수형 인터페이스로 한다.

## 기본 함수형 인터페이스

```java
// 매개변수 X, 반환값 X  
Runnable runnable = () -> System.out.println("runnable");  
runnable.run();  
  
// 매개변수 X, 반환값 O  
Supplier<String> supplier = () -> "supplier";  
System.out.println(supplier.get());  
  
// 매개 변수 O, 반환값 X  
Consumer<String> consumer = (String s) -> System.out.println(s);  
consumer.accept("consumer");  
  
// 매개변수 O, 반환값 O  
Function<String, String> function = (String s) -> s;  
System.out.println(function.apply("function"));  
  
// 매개변수 O, 반환값 Boolean  
Predicate<String> predicate = (String s) -> true;  
System.out.println(predicate.test("") ? "predicate" : "XXXXXX");  
  
// 매개변수 2개, 반환값 X  
BiConsumer<String, String> biConsumer =  
        (s1, s2) -> System.out.println(s1 + s2);  
biConsumer.accept("bi", "consumer");  
  
// 매개변수 2개, 반환값 Boolean  
BiPredicate<String, String> biPredicate =  
        (s1, s2) -> s1.equals(s2);  
System.out.println(biPredicate.test("bi", "bi") ? "bipredicate" : "XXXX");  
  
// 매개변수 2개, 반환값 1개  
BiFunction<String, String, String> biFunction = (s1, s2) -> s1 + s2;  
System.out.println(biFunction.apply("bi", "function"));  
  
// 매개변수 1개, 반환값 1개, 타입 일치  
UnaryOperator<String> unaryOperator = (s -> "unary" + s);  
System.out.println(unaryOperator.apply("operator"));  
  
// 매개변수 2개, 반환값 1개, 타입 일치  
BinaryOperator<String> binaryOperator = (s1, s2) -> s1 + s2;  
System.out.println(binaryOperator.apply("binary", "operator"));
```

자바에서 기본적으로 제공하는 함수형 인터페이스이다.  
매개변수가 3개가 필요하는 등 추가적인 작업에 대해서는 별도로 구현하자.  

> 외우기 보다는 원리를 이해하고 그때마다 사용할 것을 골라서 사용 후 코드를 정리하는 방향으로 진행

### Function Default Method

```java
Function<String, Integer> fun1 = s -> Integer.parseInt(s, 16);  
Function<Integer, String> fun2 = i -> Integer.toBinaryString(i);  
  
Function<String, String> fun3 = fun1.andThen(fun2);  
Function<String, String> fun4 = fun2.compose(fun1);  
  
System.out.println(fun3.apply("12"));  
System.out.println(fun4.apply("12"));
```

두 함수를 합성

### Predicate Default Method

```java
Predicate<Integer> p = i -> i < 100;  
Predicate<Integer> q = i -> i < 200;  
Predicate<Integer> r = i -> i % 2 == 0;  
  
// 두 Predicate를 하나로 결합할 수 있다.  
// and(), or(), negate()가 있음  
Predicate<Integer> and = p.and(r);  
  
System.out.println(and.test(100));  
System.out.println(and.test(10));
```

`and`, `or`, `not` 과 같은 집합 개념을 제공

## 컬렉션 프레임워크와 함수형 인터페이스

> 자주 반복적으로 사용하는 것들 위주로 익숙해지기

### Collection / Iterable 계열

| 인터페이스        | 메서드              | 함수형 인터페이스      | 설명             |
| ------------ | ---------------- | -------------- | -------------- |
| `Iterable`   | `forEach`        | `Consumer<T>`  | 각 요소에 대해 동작 수행 |
| `Collection` | `removeIf`       | `Predicate<T>` | 조건을 만족하는 요소 제거 |
| `Collection` | `stream`         | `Function` 계열  | Stream API 진입점 |
| `Collection` | `parallelStream` | `Function` 계열  | 병렬 스트림 생성      |
### List 계열

| 인터페이스  | 메서드          | 함수형 인터페이스          | 설명             |
| ------ | ------------ | ------------------ | -------------- |
| `List` | `replaceAll` | `UnaryOperator<T>` | 모든 요소를 변환하여 치환 |
| `List` | `sort`       | `Comparator<T>`    | 정렬 기준 제공       |
### Map 계열

| 인터페이스 | 메서드                | 함수형 인터페이스           | 설명              |
| ----- | ------------------ | ------------------- | --------------- |
| `Map` | `forEach`          | `BiConsumer<K,V>`   | 키-값 순회          |
| `Map` | `computeIfAbsent`  | `Function<K,V>`     | 키가 없을 때 값 계산    |
| `Map` | `computeIfPresent` | `BiFunction<K,V,V>` | 키가 있을 때 값 재계산   |
| `Map` | `compute`          | `BiFunction<K,V,V>` | 무조건 재계산         |
| `Map` | `merge`            | `BiFunction<V,V,V>` | 기존 값과 병합        |
| `Map` | `replaceAll`       | `BiFunction<K,V,V>` | 모든 값 일괄 변환      |
| `Map` | `getOrDefault`     | —                   | 기본값 반환 (람다는 아님) |
### Set / Queue 계열 (간접 사용)

| 인터페이스   | 메서드        | 함수형 인터페이스      | 설명         |
| ------- | ---------- | -------------- | ---------- |
| `Set`   | `removeIf` | `Predicate<T>` | 조건 기반 제거   |
| `Queue` | `removeIf` | `Predicate<T>` | 큐에서도 동일 동작 |
### Stream API (컬렉션과 가장 밀접)

| 스트림 메서드                 | 함수형 인터페이스               | 설명      |
| ----------------------- | ----------------------- | ------- |
| `filter`                | `Predicate<T>`          | 조건 필터링  |
| `map`                   | `Function<T,R>`         | 요소 변환   |
| `flatMap`               | `Function<T,Stream<R>>` | 스트림 평탄화 |
| `forEach`               | `Consumer<T>`           | 요소 소비   |
| `anyMatch` / `allMatch` | `Predicate<T>`          | 조건 검사   |
| `reduce`                | `BinaryOperator<T>`     | 요소 집계   |
| `collect`               | `Collector`             | 결과 수집   |
### 함수형 인터페이스 역할 요약

| 인터페이스               | 역할               |
| ------------------- | ---------------- |
| `Consumer<T>`       | 입력 O, 반환 X       |
| `Predicate<T>`      | 입력 O, boolean 반환 |
| `Function<T,R>`     | 입력 → 출력          |
| `UnaryOperator<T>`  | 동일 타입 변환         |
| `BiFunction<T,U,R>` | 두 입력 → 출력        |
| `BinaryOperator<T>` | 동일 타입 두 개 → 하나   |

## 기본형을 사용하는 함수형 인터페이스

- `에이To비Function` : 입력이 A 타입, 출력이 B타입 (예시 : `DoubleToIntFunction`)
- `To에이Function<T>` :  입력이 제네릭 타입, 출력이 A 타입
- `에이Function<T>` : 입력이 A 타입, 출력이 제네릭 타입
- `Obj에이Function<T>` : 입력이 제네릭과 A, 출력 X

> 기본형 대신 Wrapper 클래스를 사용하는 것은 비효율적이기에 기본형을 사용하는 함수형 인터페이스를 제공한다.

## 함수형 인터페이스 활용 방식

### 특정 로직을 예외 발생시 반복 요청

```java

package org.example.standard.lambda;  
  
import java.io.BufferedReader;  
import java.io.IOException;  
import java.io.InputStreamReader;  
import java.util.function.Function;  
import java.util.function.Supplier;  
import org.example.Console;  
  
public class RepeatExceptionEx {  
    public static void main(String[] args) throws IOException {  
  
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));  
  
//        while (true) {  
//            try {  
//                System.out.println("숫자 입력");  
//                String a = br.readLine();  
//                String b = br.readLine();  
//                int result = sum(a,b);  
//                System.out.println(result);  
//                break;  
//            } catch (Exception e) {  
//                System.out.println("다시 입력");  
//            }  
//        }  
  
        Integer result = repeatRun(() -> {  
            System.out.println("숫자 2개를 입력하시오");  
            String a = Console.readLine();  
            String b = Console.readLine();  
            return sum(a, b);  
        });  
  
        System.out.println(result);  
  
    }  
  
    static public int sum(String a, String b) {  
        return Integer.parseInt(a) + Integer.parseInt(b);  
    }  
  
    static public <T> T repeatRun(Supplier<T> supplier) {  
        while (true) {  
            try {  
                return supplier.get();  
            } catch (Exception e) {  
                System.out.println("에러 발생, 재실행");  
            }  
        }  
    }  

	// 나중에 해당 메서드 또한 다시 점검해보자,
	static public <T> void inputRepeat(Consumer<T> consumer, T params) {  
	    while (true) {  
	        try {  
	            consumer.accept(params);  
	            return;  
	        } catch (IllegalArgumentException e) {  
	            System.out.println(e.getMessage());  
	        }  
	    }  
	}

}

```