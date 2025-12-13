
# 에러와 예외

- 예러 : 프로그램 코드에 의해서 수습될 수 없는 심각한 오류
- 예외 : 프로그램 코드에 의해서 수습될 수 있는 다소 미약한 오류


# 예외 계층 구조

자바에서 모든 예외와 오류는 `Throwable` 클래스의 하위 클래스이다.  

![Pasted image 20251213120224](../../GALLERY/Pasted%20image%2020251213120224.png)

## 자바 예외의 종류

자바는 다양한 클래스 라이브러리와 관련된 여러 유형의 예외를 정의한다.  
또한 사용자가 자체 예외를 정의할 수 있다.  
 
![Pasted image 20251213120452](../../GALLERY/Pasted%20image%2020251213120452.png)

### 내장 예외 (Built-in Exception)

내장 예외는 프로그램 실행 중 발생하는 일반적인 오류를 처리하기 위해 Java가 미리 정의한 클래스이다.  
내장 예외에는 크게 2가지가 존재한다.

- 검사 예외
- 미확인 예외

검사 예외와 미확인 예외에 대한 자세한 정보는 [여기](Java-검사-예외와-비검사-예외.md)를 참조

### 사용자 정의 예외 (User-Defined Exception)

보통 `Exception`, 혹은 상황에 맞는 예외를 상속하여 구현

```java
class MyException extends Exception {  
    public MyException(String message) {  
        super("나만의 에러 메세지");  
    }  
}
```

> 필요에 따라 `Exception`이 아닌 `RuntimeException`을 상속 받아 예외처리를 선택적으로 할 수 있도록 구현하는 방향성도 고려하자.

# 예외 처리

### 정의
프로그램 실행 시 발생할 수 있는 예외의 발생에 대비한 코드를 작성하는 것

### 목적
프로그램의 비정상 종료를 막고, 정상적인 실행상태를 유지하는 것

## try-catch

- `try` 블록에는 예외를 발생시킬 수 있는 코드가 포함
- `catch` 블록은 예외가 발생할 경우 이를 처리

```java
class Main{
    public static void main(String[] args) {
        
        int n = 10;
        int m = 0;

        try {
            int ans = n / m;
            System.out.println("Answer: " + ans);
        } catch (ArithmeticException e){
            System.out.println("Error: 0으로 나눌 수 없음!");
        } 
    }
}
```

## finally

- `finally` 블록은 예외 발생 여부와 관계없이 항상 실행  
- `try-catch` 블록 다음에 반드시 **실행되어야 하는 코드를 실행**하는 데 사용됩니다.
- 데이터베이스 연결, 열려 있는 파일 및 네트워크 연결과 같은 리소스를 닫는 데 주로 사용 

```java
class Main{
    public static void main(String[] args) {
        
        int n = 10;
        int m = 0;

        try {
            int ans = n / m;
            System.out.println("답 : " + ans);
        } catch (ArithmeticException e){
            System.out.println("Error: 0으로 나눌 수 없음!");
        } finally {
	        System.out.println("코드 종료 뀨");
        }
    }
}
```

## 예외 발생

키워드 `throw`를 사용하여 명시적으로 예외를 발생시킬 수 있다.

```java
// 예외 객체 생성
Exception e = new Exception("킹부러 에러 발생");
// 예외 발생
throw e;
```

```java
// 한줄 처리
throw new Exception("킹부러 에러 발생");
```

예외 객체 생성자의 매개변수에 `String` 값을 넣어주면 해당 `String`이 `Exception` 인스턴스의 메세지로 저장

## 메서드에 예외 선언

메서드 선언부에 키워드 `thorws`를 사용하여 메서드 내에서 발생할 수 있는 예외를 작성한다.  

> `throw`와 `throws` 구분 주의!

```java
void method() throws Exception1, Exception2, Exception3 {
}
```

> 선언부에 예외를 명시하는 것은 더욱 견고한 코드를 작성하는데 도움을 준다.  
> 예를 들어 해당 코드를 상속에 상속을 이어 한다고 했을 때 해당 메서드에서 발생 할 수 있는 예외를 미리 알려주는 가이드라인 역할을 해줄 수 있기 때문이다.

```java
void 술퍼먹기() throws 종점까지잠들기, 물건잃어버리기, 다음날속뒤집어지기 {
}
```

`throws`는 직접적으로 예외를 처리하는 것이 아닌 자신을 호출한 메서드에게 예외를 전달하여 예외처리를 떠넘기는 것이다.

## 예외 출력 방식

- [`printStackTrace()`](https://www.geeksforgeeks.org/java/throwable-printstacktrace-method-in-java-with-examples/) : 예외 발생 당시 호출 스택에 있었던 메서드의 정보와 예외 메세지를 화면에 출력한다.
- [`getMessage()`](https://www.geeksforgeeks.org/java/throwable-getmessage-method-in-java-with-examples/)  : 발생한 예외 클래스의 인스턴스에 저장된 메세지를 얻는다.
- [`toString()`](https://www.geeksforgeeks.org/java/throwable-tostring-method-in-java-with-examples/) : 예외 정보를 예외 이름 형식으로 출력합니다.

> `printStackTrace(PrintStream s)`와 같이 사용하면 발생한 예외에 대한 정보를 파일에 기록할 수 있다.

## 예외 되던지기

- 예외를 처리한 후에 다시 예외를 발생시키는 것
- 호출한 메서드와 호출된 메서드 양쪽 모두에서 예외처리하는 것

```java
public class ExceptionRethrow {  
    public static void main(String[] args) {  
        try {  
            method();  
        } catch (Exception e) {  
            System.out.println("에러 2번 처리");  
        }  
    }  
  
    static void method() throws Exception {  
        try {  
            throw new Exception();  
        } catch (Exception e) {  
            System.out.println("에러 1번 처리");  
            throw e;  
        }  
    }  
}
```

대표적인 예시가 Spring Framework의 [`@ControllerAdvice`](../SPRING/Spring-ControllerAdvice.md)와 `@ExceptionHandler` 어노테이션을 같이 사용할때이다.

## 연결된 예외

- 한 예외가 다른 예외를 발생시킬 수 있다.
- 예외 A가 예외 B를 발생시키면, A는 B의 원인 예외

### 사용 목적

- 여러 예외를 하나로 묶어서 다루기 위해
- checked예외를 unchecked예외로 변경하려고 할 때

```java
public class ChainedException1 {  
    public static void main(String[] args) {  
        try {  
            study();  
        } catch (StudyException e) {  
            e.printStackTrace();  
        } catch (Exception e) {  
            throw new RuntimeException(e);  
        }  
    }  
  
    static void study() throws StudyException {  
        try {  
            System.out.println("공부 시작");  
            throw new PlayGameException();  
        } catch (PlayGameException e) {  
            StudyException se = new StudyException("공부 중 예외 발생");  
            se.initCause(e);  
            throw se;  
        }  
    }  
}  
  
  
class StudyException extends Exception {  
  
    public StudyException(String message) {  
        super(message);  
    }  
  
    @Override  
    public synchronized Throwable initCause(Throwable cause) {  
        return super.initCause(cause);  
    }  
}  
  
class PlayGameException extends Exception {  }  
  
class SleepException extends Exception {  }
```


## try-with-resource

자세한 내용은 [여기](Java-try-with-resource.md)를 참조

# 출처
[geeksforgeeks](https://www.geeksforgeeks.org/java/exceptions-in-java/)

