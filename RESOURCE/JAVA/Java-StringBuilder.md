# StringBuilder

`stringBuilder` 클래스는 변경 가능한 문자열 시퀀스를 제공한다.  
불변 객체인 `String` 클래스와 달리, `StringBuilder`는 새로운 객체를 생성하지 않고도 문자열 시퀀스를 수정할 수 있으므로, 빈번한 문자열 연산에서 메모리 효율성이 높고 속도가 빠르다.  

- StringBuffer와 유사한 기능을 제공하지만 스레드 안전성은 보장되지 않습니다.
- StringBuilder는 동기화되지 않으므로 단일 스레드 애플리케이션에서 성능이 더 좋습니다.
- 스레드 안전성이 요구되는 경우에만 StringBuffer를 사용하고, 그렇지 않은 경우에는 성능 향상을 위해 StringBuilder를 사용하는 것이 좋습니다.

## 선언

```java
StringBuilder sb = new StringBuilder("초기 문자열");
```

## 예시

```java
public class StringBuilder_Ex {
    public static void main(String[] args)  {
        String s = "abcdefg";
        StringBuilder sb = new StringBuilder(s); // String -> StringBuilder
		
        System.out.println("처음 상태 : " + sb); //처음상태 : abcdefg
        System.out.println("문자열 String 변환 : " + sb.toString()); //String 변환하기
        System.out.println("문자열 추출 : " + sb.substring(2,4)); //문자열 추출하기
        System.out.println("문자열 추가 : " + sb.insert(2,"추가")); //문자열 추가하기
        System.out.println("문자열 삭제 : " + sb.delete(2,4)); //문자열 삭제하기
        System.out.println("문자열 연결 : " + sb.append("hijk")); //문자열 붙이기
        System.out.println("문자열의 길이 : " + sb.length()); //문자열의 길이구하기
        System.out.println("용량의 크기 : " + sb.capacity()); //용량의 크기 구하기
        System.out.println("문자열 역순 변경 : " + sb.reverse()); //문자열 뒤집기
        System.out.println("마지막 상태 : " + sb); //마지막상태 : kjihgfedcba
    }
}

```

<br>

# 출처

[geeksforgeeks](https://www.geeksforgeeks.org/java/stringbuilder-class-in-java-with-examples/)
[티스토리](https://coding-factory.tistory.com/546)