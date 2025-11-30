메서드의 매개변수 개수를 동적으로 지정하는 방법이다
## 사용 방식
```java
public void 함수(타입 ...변수명) {
}
```
### 가변인자를 매개 변수 중 가장 마지막에 사용
```java
public void func(String ...str, int a) {

}
```
`func(a,b,c,d)` 이런식으로 호출될탠데 어디서부터 어디까지가 가변인자인지 모름
### 같은 형식의 매개변수가 존재하는 오버로딩
```java
public void func(String str1, String ...str2) {
	
}
public void func(String ...str2) {

}
```
오버로딩 또한 문제 발생
## 사용 이유
> 매개변수를 배열로 전달하는 것과 무슨 차이점이 있는가?

매개 변수를 생략 가능하다
## 대표적 예시
### printf()
```java
public PrintStream printf(String format, Object ...args) { ... }
```
### ErrorMessage
```java
import java.text.MessageFormat;

public enum ErrorMessage {
    ERROR("기본 에러"),
    RESOURCE_NOT_FOUND("해당 ID와 일치하는 {0} 정보를 찾지 못했습니다. (id: {1})"),
    ;

    private final String message;
    private static final String PREFIX = "[ERROR]";

    ErrorMessage(String message) {
        this.message = message;
    }

    public String getMessage() {
        return MessageFormat.format("{0} {1}", PREFIX, message);
    }

    public String format(Object... args) {
        return MessageFormat.format("{0} {1}", PREFIX, MessageFormat.format(message, args));
    }
}

```


`
```java
  private Object args = null;
  
  
public static RestApiException of(ErrorCode errorCode) {  
    return new RestApiException(errorCode);  
}  
  
public static RestApiException argsFrom(ErrorCode errorCode, Object ...args) {  
    return new RestApiException(errorCode, args);  
}
```