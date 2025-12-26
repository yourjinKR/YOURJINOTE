# 열거형 (Enum)

- 심볼릭 상수를 정의함, 관련된 상수를 하나로 묶은 것
- 컴파일시 값 & 타입 체크
- 열거형 상수는 객체이며 생성할 수 있는 객체의 수가 제한

```java
enum Week {  
    MONDAY, TUESDAY, WEDNESDAY,  
    THURSDAY, FRIDAY, SATURDAY, SUNDAY  
}
```

## 사용 이점

- 형안정성 보장
- 가독성이 좋은 코드를 제공

## 메서드 및 사용 방법

```java
enum Direction { EAST, WEST, SOUTH, NORTH }
```

> 위와 같은 `enum`을 기반으로 아래 예시 코드 작성

- `values()` : 해당 열거형 타입의 모든 상수를 배열 형태로 반환 
- `valueOf(String name)` : 문자열과 일치하는 상수를 반환

```java
Direction directionByString = Direction.valueOf("NORTH");  
System.out.println(directionByString.name()); // NORTH  
  
Direction[] values = Direction.values();  
for (Direction value : values) {  
    System.out.print(value.name() + " "); // EAST WEST SOUTH NORTH  
}  
System.out.println();
```

- `name()`

```java
Direction direction = Direction.NORTH;
String name = direction.name();  
System.out.println(name); // NORTH 
```

- `ordinal()`

```java
int ordinal = direction.ordinal();  
System.out.println(ordinal); // 3  
```

- `toString()`

```java
String string = direction.toString();  
System.out.println(string); // NORTH  
```

## 비교 연산자

```java
Direction direction = Direction.NORTH;
Direction directionByString = Direction.valueOf("NORTH");  

System.out.println(direction == directionByString); // true
```

> 열거형 상수 간에는 `==` 를 사용할 수 있다.  (상수 간 빠른 연산을 제공)
> 그러나 `<`, `>` 와 같은 비교 연산자는 사용할 수 없다.

## Switch 문과 같이 사용

```java
switch (direction) {  
    case EAST -> System.out.println("동");
    case WEST -> System.out.println("서");  
    case SOUTH -> System.out.println("남");  
    case NORTH -> System.out.println("북");  
}
```

코드의 간결성을 위해 `열거형이름.상수명`이 아닌 `상수명`만 표기하여 작성한다.

## 열거형에 멤버 추가

- `ordinal()`를 통해 상수값에 대한 순서를 반환할 수 있지만, 만약 임의의 값이 아닌 불규칙적인 특정한 값을 지정해야 할 때 각 상수에 멤버를 추가할 수 있다. 

> 해당 예시는 http 응답 코드를 간단하게 표현한 것이다.

```java
enum STATUS {  
  
    NOT_FOUND(400),  
    UNAUTHORIZED(401)  
    ;  
  
    private final int code;  
  
	// private 생략 가능
    STATUS(int code) {  
        this.code = code;  
    }  
  
    int getCode () {  
        return this.code;  
    }  
}
```

- 열거형의 생성자는 묵시적으로 `private`
- 열거형의 멤버 또한 상수로 관리하기에 `private final` 키워드를 붙이도록 권장

> `enum`은 "상수" + 타입 + 행위(메서드)를 하나의 닫힌 집합으로 묶기 위한 장치이다.  
> 그렇기에 각각의 멤버 변수를 생성자를 통해 초기화 할 수 없도록 설계한다.


```java
STATUS notFound = STATUS.NOT_FOUND;  
System.out.println(notFound); // NOT_FOUND  
System.out.println(notFound.name()); // NOT_FOUND  
System.out.println(notFound.getCode()); // 400
```

<br>

# 출처

[유튜브 - 자바의 정석](https://www.youtube.com/watch?v=qZj_nPEhnqM&list=PLW2UjW795-f4dHD3VADsvH7ZkPuFAF7Vo&index=71)  

<br>

# 다시 점검할 부분

[기술면접-Java-Enum](../../AREA/취업/기술면접/기술면접-Java-Enum.md)
