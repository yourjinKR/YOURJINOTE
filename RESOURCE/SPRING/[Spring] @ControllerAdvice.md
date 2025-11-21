## 개요
스프링에서는 기본적으로 제공하는 에러 페이지가 있다.
이는 `BasicErrorController`가 담당한다.
`BasicErrorController` 는 요청의 Accept 헤더 값이 text/html일 때, 예외가 발생하면 `/error` 라는 경로로 재요청을 보내줍니다.

해당 방식은 사용자가 에러 페이지의 경로를 안다면 에러 페이지 자체에 접근할 수 있다는 문제가 있다.
이를 해결해주는 것이 `@ExceptionHandler`이다.

# @ExceptionHandler 
```java
@RestController
public class SimpleController {
    @ExceptionHandler(value = IllegalArgumentException.class)
    public ResponseEntity<String> invokeError(IllegalArgumentException e) {
         ...
				return new ResponseEntity<>("error Message", HttpStatus.BAD_REQUEST);
    }
}
```
`@ExceptionHandler` 어노테이션을 사용하면 value로 원하는 예외를 지정하고 이를 핸들링이 가능하다.
value 속성을 지정한 예외 뿐만 아니라 자식 클래스도 전부 캐치한다.
`@ExceptionHandler`는 코드를 작성한 컨트롤러에서만 발생하는 예외만 처리한다.
그렇기에 여러 컨트롤러가 있다면 여러 handler를 만들어야 한다.

이를 해결한 것이 `@ControllerAdvice`이다.

# ControllerAdvice
`@ControllerAdvice`는 `@Component` 어노테이션의 특수한 케이스이다.
스프링 부트 애플리케이션에서 전역적으로 예외를 핸들링할 수 있게 해주는 어노테이션입니다.
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ControllerAdvice
@ResponseBody
public @interface RestControllerAdvice {
    ...
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface ControllerAdvice {
    ...
}
```
`@Component`어노테이션을 붙이면 Component Scan 과정을 거쳐 Bean으로 등록된다.
즉, 이 어노테이션을 사용하는 `ControllerAdvice` 또한 Bean으로 관리됨.
또한, `@RestControllerAdvice`는 `@ResponseBody` 어노테이션이 붙어있으므로 응답을 Json으로 처리함.
`@ControllerAdvice` 어노테이션을 통해 예외를 핸들링하는 클래스를 구현함.

```java
@ControllerAdvice
public class SimpleControllerAdvice {

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<String> IllegalArgumentException() {
        return ResponseEntity.badRequest().build();
    }
}
```
위 처럼 코드를 작성하게 되면 애플리케이션 내의 모든 컨트롤러에서 발생하는 `IllegalArgumentException`을 해당 메서드가 처리하게 된다.
## 주의점
여러 `ControllerAdvice`가 있을 때 `@Order`어노테이션으로 순서를 지정하지 않는다면 Spring은 `ControllerAdvice`를 임의의 순서로 호출한다. 즉, 사용자가 예상하지 못한 예외 처리가 발생할 수 있다.
## basePackages
만약 여러 `ControllerAdvice`를 세분화하고 싶다면 `basePackages` 속성을 이용할 수 있다.
작성된 패키지와 하위 패키지에서 발생하는 예외는 해당 `ControllerAdvice`에서 처리하도록 지정할 수 있습니다.
```java
@ControllerAdvice(basePackages = {"org.woowa.tmp.pkg"}
public class SimpleControllerAdvice {

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<String> IllegalArgumentException() {
        return ResponseEntity.badRequest().build();
    }
}
```
`basePackages` 속성을 통해 뷰와 모델을 이용해 직접 페이지를 반환하는 Controller는 예외가 발생하면 직접 예외 페이지를 응답하게 하고, JSON 형식으로 데이터를 주고받는 Controller는 예외가 발생하면 예외에 대한 정보를 JSON 형식으로 응답하게 할 수도 있습니다.

## assignableTypes
`assignableTypes` 속성을 이용하면 클래스 단위로도 `ControllerAdvice`를 적용할 수 있다.
클래스 단위로 사용되는 만큼 `basePackages` 속성보다 조금 더 세밀하게 처리를 분리해 주고 싶을 때 사용하면 유용하다.
```java
@ControllerAdvice(assignableTypes = {SimpleController.class}
public class SimpleControllerAdvice {

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<String> IllegalArgumentException() {
        return ResponseEntity.badRequest().build();
    }
}
```
## ResponseEntityExceptionHandler
Spring은 스프링 예외를 미리 처리해 둔 `ResponseEntityExceptionHandler`를 추상 클래스로 제공하고 있습다.
`ResponseEntityExceptionHandler`에는 스프링 예외에 대한 `ExceptionHandler`가 모두 구현되어 있다.
하지만 에러 메시지는 반환하지 않으므로 스프링 예외에 대한 에러 응답을 보내려면 이 클래스를 상속하여 아래 메서드를 오버라이딩 해야 한다.
```java
public abstract class ResponseEntityExceptionHandler {
    ...

    protected ResponseEntity<Object> handleExceptionInternal(
			Exception ex, @Nullable Object body, HttpHeaders headers, HttpStatus status, WebRequest request) {

		if (HttpStatus.INTERNAL_SERVER_ERROR.equals(status)) {
			request.setAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE, ex, WebRequest.SCOPE_REQUEST);
		}
		return new ResponseEntity<>(body, headers, status);
	}
}
```

# 출처
https://tecoble.techcourse.co.kr/post/2023-05-03-ExceptionHandler-ControllerAdvice/
