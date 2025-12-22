# EnumSet

Java에서 [열거형](Java-열거형-Enum.md)과 함께 사용하기 위한 특수한 구현체이다.  

- `AbstractSet` 클래스를 확장하고 [[RESOURCE/JAVA/Java-Set.md|Set]] 인터페이스를 구현함
- `HashSet`보다 훨씬 고성능
- EnumSet의 모든 요소는 생성할때 명시적 또는 암시적으로 지정된 단일 열거형 유형에서 가져와야 함 
- `null`를 허용하지 않으며 `null` 삽입 시도시 NPE 발생
- fail-safe iterator를 사용하며 컬렉션을 순회하는 동안 컬렉션이 수정되더라도 예외가 발생하지 않음
	- 일반적인 컬렉션은 `ConcurrentModificationException`가 발생함

```java
// AbstractSet이기에 new 연산을 통한 객체 생성이 불가함  
  
// 1. 특정 요소만 담기  
EnumSet<Style> styles = EnumSet.of(Style.BOLD, Style.ITALIC);  
  
// 2. 모든 요소 담기  
EnumSet<Style> allStyles = EnumSet.allOf(Style.class);  
  
// 3. 비어있는 셋 만들기  
EnumSet<Style> emptyStyles = EnumSet.noneOf(Style.class);  
  
// 4. 범위 지정 (BOLD부터 UNDERLINE까지)  
EnumSet<Style> rangeStyles = EnumSet.range(Style.BOLD, Style.UNDERLINE);
```

<br>

# 출처

[geeksforgeeks](https://www.geeksforgeeks.org/java/enumset-class-java/)  
