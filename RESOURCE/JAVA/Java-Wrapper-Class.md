# Wrapper Class

- 기본 데이터 형식은 클래스가 아니므로 메소드를 가지지 못함
- 이 문제를 해결하기 위해 기본 데이터 형식을 클래스로 만드는 것이 Wrapper Class
	- Byte, Character
	- Short
	- Integer, Long
	- Float, Double
	- Boolean

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
자바 9버전 부터 `@Deprecated`

## AutoBoxing과 AutoUnBoxing

```java
Integer num1 = 123; // 자동 박싱
int num2 = num1; //자동 언박싱
```

위와 같이 박싱과 언박싱이 자동으로 발생하기 때문이다.

# 출처
[인프런 - 널널한 개발자](https://www.inflearn.com/course/%EB%8F%85%ED%95%98%EA%B2%8C-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-java-part2/dashboard)

[티스토리 - 래퍼 클래스란 무엇인가](https://coding-factory.tistory.com/547)