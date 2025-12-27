
# 람다

- 메서드를 하나의 식으로 표현한 것
- 람다식은 함수형 인터페이스의 구현체로서 익명 함수를 간결하게 표현하는 문법

> 메서드를 람다식으로 표현하면 메서드의 이름과 반환값이 없어지므로, 람다식을 **"익명 함수"** 라고도 부른다.

## 작성 방식

```java
// 기본 메서드 작성 방식
int max(int a, int b) {  
    return a > b ? a : b;  
}

(int a, int b) -> a > b ? a : b;
```

메서드에서 이름과 반환 타입을 제거 후 매개변수 선언부와 메서드 선언부 사이에 `->`를 넣는다.

자바에서는 메서드 호출을 위해서는 객체를 생성해야 하기에 불편함이 있었으나 이를 람다식으로 해결할 수 있다.

> `static` 키워드를 통해 정적 메서드를 만들 수 있지만, 해당 로직이 사용되는 빈도 수가 적다면 리소스 낭비가 될 수 있다.

### return 생략

```java
(int a, int b) -> {  
    return a > b ? a : b;  
};

(int a, int b) -> a > b ? a : b;
```

`return` 대신 식을 대신 할 수 있기에 위와 같이 `return`과 `{}`를 생략 가능

### 타입 생략

```java
(a, b) -> a > b ? a : b;
```

선언된 매개변수의 타입이 추론이 가능한 경우는 생략 가능하다.  
람다식 대부분은 [타입 추론](Java-타입-추론.md)이 가능하다.

> 람다식의 대부분은 왜 타입 추론이 가능한가?


## 함수형 인터페이스

자바에서 모든 메서드는 클래스 내에 포함되어야 한다.  
즉, 람다식 또한 객체이며 아래와 같이 익명 클래스의 객체와 동등하다.

```java
IntBinaryOperator intBinaryOperator =  
        (int a, int b) -> a > b ? a : b;
```

```java
Object intBinaryOperatorByObject = new Object() {  
    int max(int a, int b) {  
        return a > b ? a : b;  
    }  
};
```

자세한 내용은 [여기](Java-Functional-Interface-함수형-인터페이스.md)를 참고

## 메소드 참조

```java
클래스이름::메서드이름
참조변수::메서드이름
```

람다식이 하나의 메서드만 호출하는 람다식은 메서드 참조로 간단히 할 수 있다.

```java
// 전
Consumer<String> consumer = s -> System.out.println(s);  
consumer.accept("유어진");  
  
// 후
Consumer<String> methodReference = System.out::println;  
methodReference.accept("유어진");
```

> 람다식의 일부가 생략됐음에도 불구하고, 컴파일러는 함수형 인터페이스의 지정된 제네릭 타입을 통해 해당 메서드가 어떤 값들을 매개변수로 받을지 알게 된다.

```java
BiConsumer<String, String> biMethodRef = MethodReferenceEx::printBi;  
biMethodRef.accept("유", "어진");

static public void printBi(String s1, String s2) {  
    System.out.println(s1 + s2);  
}
```