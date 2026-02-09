# 개요

- Checked Exception (검사 예외) : 컴파일 시점에 검사되므로 프로그래머가 명시적으로 처리
- UnChecked Exception (비검사 예외) : 런타임에 검사되므로 컴파일 시점에 명시적으로 처리

![Pasted image 20251213120452](../../GALLERY/Pasted%20image%2020251213120452.png)

# Java의 검사 예외

> `Exception`을 상속하면서 `RuntimeException`을 상속하지 않으면 **검사 예외**

- **컴파일 시점에 반드시 처리해야 하는 예외**
- `try-catch`로 직접 처리하거나 `throws`로 호출한 쪽이 위임하지 않으면 컴파일 에러 발생
- 예외 발생 가능성이 예측 가능하고 외부 요인에 의해 발생하는 경우가 많음

```java
public void readFile() throws IOException {
    FileReader fr = new FileReader("test.txt");
}
```

### 대표 예시

- `IOException`
- `SQLException`
- `ClassNotFoundException`
- `FileNotFoundException`

# Java의 비검사 예외

> `RuntimeException`을 상속한다면  **비검사 예외**

- 컴파일 시 강제되지 않는 에외
- 예외 처리를 하지 않아도 컴파일은 정상 작동
- 런타임에 발생
- 주로 프로그래머의 실수이며 사전에 방지해야 할 논리 오류

```java
String s = null;
System.out.println(s.length()); // NullPointerException
```

### 대표 예시

- `NullPointerException`
- `ArrayIndexOutOfBoundsException`
- `IllegalArgumentException`
- `ArithmeticException`


# 사용 구분

호출하는 쪽에서 처리할 수 있다면 검사 예외를 사용하고 그렇지 않다면 비검사 예외를 사용하자



# 출처

[geeksforgeeks](https://www.geeksforgeeks.org/java/java-checked-vs-unchecked-exceptions/)  
[baeldung](https://www.baeldung.com/java-checked-unchecked-exceptions)

