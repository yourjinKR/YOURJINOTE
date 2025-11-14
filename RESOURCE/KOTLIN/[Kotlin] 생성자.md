[출처1](https://manorgass.tistory.com/79)
[출처2](https://enter.tistory.com/227)
# 생성자
## 생성자의 역할
- 인스턴스의 속성을 초기화
- 인스턴스 생성시 구문을 수행
생성자는 init과 constructor 두 가지 함수를 이용해 초기화 할 수 있음

## 자바 생성자
```java
class Person {
	private String name;
	private int age;
	
	public Person(String name, int age) {
		this.name = name;
		this.age = age;
	}
}
```
자바에서는 다음과 같이 생성자를 작성한다.
## 코틀린 생성자
### constructor 함수
```kotlin
class Person {  
    constructor(name: String, age: Int) {  
        this.name = name  
        this.age = age  
    }  
    var name: String = ""  
    var age: Int = 0  
}  
```
### 기본 생성자 선언
```kotlin
// 클래스명 옆에 기본 생성자를 선언 가능함    
class Person2 constructor(name: String, age: Int) {  
    var name: String = name  
    var age: Int = age  
}  
```

```kotlin
// constructor 생략 가능  
class Person3(name: String, age: Int) {  
    var name: String = name  
    var age: Int = age  
}  
```

```kotlin
// 내부에서 프로퍼티 선언 가능  
class Person4(var name: String, var age: Int) {  
}  
```

### 보조 생성자
this를 사용하여 기본 생성자를 호출하여 초기화
```kotlin
// 이런식으로 생성자 지정 가능  
class Person5(name: String, age: Int) {
	// 보조 생성자, this를 사용하여 기본 생성자를 호출하여 초기화
    constructor(name: String) : this(name, age = 0) {  
        this.name = name  
    }  
    var name: String = name  
    var age: Int = age  
}  
```
### 총정리
```kotlin
// 일반적인 생성자 방식  
class Person {  
    constructor(name: String, age: Int) {  
        this.name = name  
        this.age = age  
    }  
    var name: String = ""  
    var age: Int = 0  
}  
  
// 클래스명 옆에 기본 생성자를 선언 가능함  
class Person2 constructor(name: String, age: Int) {  
    var name: String = name  
    var age: Int = age  
}  
  
// constructor 생략 가능  
class Person3(name: String, age: Int) {  
    var name: String = name  
    var age: Int = age  
}  
  
// 내부에서 프로퍼티 선언 가능  
class Person4(var name: String, var age: Int) {  
}  
  
// 이런식으로 생성자 지정 가능  
class Person5(name: String, age: Int) {  
    constructor(name: String) : this(name, age = 0) {  
        this.name = name  
    }  
    var name: String = name  
    var age: Int = age  
}  
  
// 기본값 설정  
class Person6(var name: String = "개발자", var age: Int = 0) {}  
  
fun main() {  
    val p51 = Person5("유어진")  
    val p52 = Person5("유어진", 26)  
  
    val p61 = Person6()  
    val p62 = Person6("유어진")  
    val p63 = Person6(age = 26)  
}
```
여기서 주목할 점은 **주 생성자를 지정**할 수 있을 뿐만 아니라 **기본값도 설정**할 수 있다는 것이다 