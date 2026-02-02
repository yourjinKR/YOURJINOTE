# Wrapper Class

기본 데이터 형식은 클래스가 아니므로 메소드를 가지지 못한다.  
이 문제를 해결하기 위해 기본 데이터 형식을 클래스로 만드는 것이 Wrapper Class이다.

| **분류** | **기본자료형 타입(Primitive Type)** | **래퍼 클래스(Wrapper Class)** |
| ------ | ---------------------------- | ------------------------- |
| 진위형    | boolean                      | Boolean                   |
| 문자형    | char                         | **Character**             |
| 정수형    | byte                         | Byte                      |
| 정수형    | short                        | Short                     |
| 정수형    | int                          | **Integer**               |
| 정수형    | long                         | Long                      |
| 실수형    | float                        | Float                     |
| 실수형    | double                       | Double                    |

## Primitive Data Type과 Wrapper Class의 비교 

| **분류**                  | **기본 자료형**          | **래퍼 클래스**                |
| ----------------------- | ------------------- | ------------------------- |
| 사용 목적                   | 간단하고 빠르게 처리가 필요한 경우 | 객체로 다루어야 하는 경우            |
| 값 비교 방법                 | `==` 연산자를 이용하여 비교한다 | `equals()` 메소드를 사용하여 비교한다 |
| 처리 속도                   | 빠르다                 | 느리다                       |
| 메모리 사용량                 | 작다                  | 크다                        |
| **null 초기화 가능 여부**      | 불가능                 | 가능                        |
| 외부에서 변경 가능 여부           | 가능                  | 불가능                       |
| 숫자의 산술연산 가능 여부          | 가능                  | 불가능                       |
| **제너릭 타입 내에서 사용 가능 여부** | 불가능                 | 가능                        |

# Boxing과 UnBoxing

```java
Integer num1 = new Integer(123); // 박싱
int num2 = num1.intValue(); // 언박싱
```
- 박싱 : 기본 타입의 값을 포장 객체로 만드는 과정
- 언박싱 : 포장 객체를 기본 타입의 값을 얻어내는 과정

```java
@Deprecated(since="9", forRemoval = true)  
public Integer(int value) {  
    this.value = value;  
}
```
Java 1.5 이상 부터 **오토 언박싱**으로 별도의 수동적 처리 없이 박싱되었다.  
그 이후 자바 9버전 부터 `@Deprecated` 되었다.

## AutoBoxing과 AutoUnBoxing

```java
Integer num1 = 123; // 오토 박싱
int num2 = num1; // 오토 언박싱
```

Java 1.5 이상 버전에서 지원되는 기능이다.  
래퍼 클래스의 인스턴스에 저장된 값을 다시 기본 타입을 꺼내는 과정을 자동으로 처리해주는 기능이다.

# 출처
[인프런 - 널널한 개발자](https://www.inflearn.com/course/%EB%8F%85%ED%95%98%EA%B2%8C-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-java-part2/dashboard)

[티스토리 - 래퍼 클래스란 무엇인가](https://coding-factory.tistory.com/547)