
# 어노테이션

주석처럼 프로그래밍 언어에 영향을 미치지 않으며, 유용한 정보를 제공한다.

## 표준 어노테이션

JDK에서 기본적으로 제공하는 어노테이션

### @Override

오버라이딩 올바르게 했는지 컴파일러가 체크

> 만약 재정의할 메서드의 이름을 잘못 표기했을 때 해당 메서드가 정상적으로 오버라이딩 됐는지 확인할 수 없다. 그러나 `@Override` 어노테이션을 붙인다면 컴파일러가 부모의 메서드 중 특정 이름을 가진 메서가 있는지 확인하고 없다면 에러 메세지를 출력한다.

### @Deprecated

앞으로 사용하지 않을 것을 권장하는 필드나 메서드를 명시

> 버전업에 따라 Java가 기존 기능을 대체할 새로운 기능이 나오기도 한다.  
> 기존의 API를 삭제하는 것이 아닌 `@Deprecated` 어노테이션을 통해  이전 버전의 호환성을 유지한다.

### @FunctionalInterface

함수형 인터페이스임을 컴파일러가 체크

> 함수형 인터페이스는 단 하나의 추상 메서드만을 가진다.

### @SuppressWarnings

컴파일러의 경고 메세지가 나타나지 않게 억제한다.

### @SafeVarargs

매개변수가 가변인자일 때, 발생하는 unchecked 경고를 억제한다.

### 메타 어노테이션

어노테이션을 만들 때 사용하는 어노테이션이다.  
자세한 내용은 [여기](Java-Meta-Annotation-메타-어노테이션.md)를 참고.

## 어노테이션 타입 정의  

```java
@interface 어노테이션이름 {  
    타입 요소이름();  
}
```

**어노테이션에 선언된 메서드**가 **어노테이션의 요소**가 된다.  

> 선언된 메서드가 없는, 요소를 하나로 정의하지 않은  어노테이션은 **마커 어노테이션**이라고 한다.

어노테이션의 요소는 **추상 메서드**의 형태를 가진다.

> 어노테이션은 인터페이스처럼 상수는 정의할 수 있으나, 디폴트 메서드는 정의할 수 없다.

어노테이션 작성시 모든 요소를 지정해야 한다.  
그렇기 때문에 편의성을 위해 `default` 값을 지정한다.

### java.lang.annotation.Annotation

- 모든 어노테이션의 조상
- `Object`와 같이 `equals()`, `hashCode()`, `toString()`과 같은 메서드를 호출 가능

```java
public interface Annotation {
	boolean equals(Object obj);
	int hashCode();
	String toString();
	Class<? extends Annotation> annotationType();
}
```

### 요소 선언시 규칙

- 요소의 타입은 기본형, String, enum, 어노테이션, Class 만 허용한다.
- () 안에 매개변수를 선언할 수 없다.
- 예외를 선언할 수 없다.
- 요소를 타입 매개변수로 정의할 수 없다.

### 요소 기본값 지정

```java
@Repeatable(ToDodos.class)  
@interface Todo{  
    String value();  
    String detail() default "";  
}

@Todo(value = "자바 어노테이션 공부하기")  
@Todo(  
        value = "개인 프로젝트 PR 1개 커밋",   
        detail = "유저 프로필 조회 API 구현"  
)  
class Today { }
```

![Pasted image 20251226170459](../../GALLERY/Pasted%20image%2020251226170459.png)

기본값은 `null`이 될 수 없으며 반드시 상수여야 한다.

### 요소명 생략

```java
@interface WillBe {  
    String value() default "성공함";  
}

@WillBe  
static class ToDo2026 {}
```

요소명이 `value` 이고 `default` 값이 있다면 생략 가능하다.

### 값에 배열

한 요소에 여러 값들이 요구된다면 배열을 통해 관리할 수 있다.

```java
@interface MyGoal {  
    String[] value() default {"취업", "연애", "가족의 행복"};  
}

@MyGoal(value = { "취업", "가족의 행복" })  
static class ToDo2026 {}
```

<br>

# 출처

[유튜브 - 자바의 정석](https://www.youtube.com/watch?v=_vd7s1LmYgw&list=PLW2UjW795-f4dHD3VADsvH7ZkPuFAF7Vo&index=74)  

<br>

# 다시 점검할 부분

[자바 어노테이션에 대해 설명해주세요](자바%20어노테이션에%20대해%20설명해주세요.md)
