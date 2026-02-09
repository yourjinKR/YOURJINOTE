가장 큰 차이는 **컴파일러가 예외 처리를 강제하는지 여부**입니다.  

Checked Exception은 `RuntimeException`을 상속하지 않는 예외들로, 컴파일러가 `try-catch`로 잡거나 `throws`로 선언했는지 **컴파일 타임에 검사**합니다. 주로 복구 가능한 외부 상황(예: I/O 오류)에 사용됩니다.

반면, **Unchecked Exception**은 `RuntimeException`과 `Error`를 포함하며, 컴파일러가 예외 처리를 강제하지 않습니다. 이는 주로 프로그램 로직 오류나 복구 불가능한 시스템 오류를 나타냅니다.

