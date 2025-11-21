# 개요

나를 포함한 팀원들의 도전은 **프로젝트 완성**과 **코틀린 학습**이 중점이었기에
뷰 템플릿은 사용해본 경험이 있는 JSP를 사용하기로 했다.

퀴즈 기능을 나와 팀원이 설정과 기록으로 구분하여 개발을 진행했다. 
구현해야 할 View가 많지 않다고 판단하여 빠른 개발을 위해
퀴즈에 대한 페이지 구현 내가 온전히 담당하고 팀원분께서는 백엔드 코드 리팩토링과 새로운 요구사항에 맞는 API를 추가 구현하도록 업무를 분업했다.

# JSP 세팅

## 설정
### build.gradle.kt 추가
```kotlin
implementation("org.apache.tomcat.embed:tomcat-embed-jasper") implementation("org.glassfish.web:jakarta.servlet.jsp.jstl:2.0.0")
```
### 파일 위치 규약
```
src
└── main
    └── webapp
        └── WEB-INF
            └── jsp
                ├── home.jsp
                └── user
                    └── profile.jsp
```
### application.yaml
```yaml
spring:  
  application:  
    name: wordCapsule  
  mvc:  
    view:  
      prefix: /WEB-INF/views/  
      suffix: .jsp
```

## 뷰 레이아웃
- 레이아웃을 header, content, footer로 나누어 최상위 레이아웃인 index.jsp에서 관리하도록 설계했다.
- 실질적인 페이지에 대한 구분은 model로 부터 받은 path의 값에 따라 main-content에 레이아웃 된다.
- 해당 구조의 장단점은 다음과 같다
	1. footer와 header가 필요한 페이지에 각각 jsp를 include할 필요 없음
	2.  모든 페이지가 하나의 파일(index.jsp)에서 다뤄지기 때문에 통일된 작업을 수행 가능
	3. 백엔드에서 요청시 path를 명시적으로 입력할 필요가 있음
### index.jsp
- 페이지의 최상위 레이아웃을 수행
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>  
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<jsp:useBean id="path" class="java.lang.String" scope="request"/>  
  
<c:set var="layout" value="${empty layoutType ? 'FULL' : layoutType.toString()}"/>  
  
<!doctype html>  
<html lang="ko">  
<head>  
    <meta charset="UTF-8">  
    <title>WordCapsule</title>  
    <%-- 공통 스타일시트 등 링크 --%>  
    <link rel="stylesheet" href="${pageContext.request.contextPath}/css/style.css">  
</head>  
<body data-login-id="${sessionScope.loginId}">  
  
<div class="app-container">  
    <c:if test="${layout eq 'FULL' or layout eq 'NO_FOOTER'}">  
        <jsp:include page="layout/header.jsp"/>  
    </c:if>  
  
    <main class="main-content">  
        <c:if test="${not empty path}">  
            <jsp:include page="${path}" flush="true"/>  
        </c:if>  
  
        <c:if test="${empty path}">  
            <jsp:include page="content/home.jsp" flush="true"/>  
        </c:if>  
    </main>  
  
    <c:if test="${layout eq 'FULL' or layout eq 'NO_HEADER'}">  
        <jsp:include page="layout/footer.jsp"/>  
    </c:if>  
</div>  
  
</body>  
</html>
```

**수정안**
일부 페이지에서는 footer와 header가 필요 없는 경우가 있었다.
`layoutType`프로퍼티를 추가하여 이에 따라 레이아웃을 규정할 수 있도록 수정했다.
### header.jsp
- 상단 내비게이션 기능을 수행
- 홈으로 이동
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<header class="app-header">
    <h1 class="title"><a href="${pageContext.request.contextPath}/">WordCapsule</a></h1>
</header>
```
### footer.jsp
- 하단 내비게이션 기능을 수행
- 퀴즈 기록, 홈, 마이페이지로 이동
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<nav class="bottom-nav">
    <a class="bottom-nav-item" href="${pageContext.request.contextPath}/quiz/records">
        <span>기록</span>
    </a>
    <a class="bottom-nav-item" href="${pageContext.request.contextPath}/">
        <span>홈</span>
    </a>
    <a class="bottom-nav-item" href="${pageContext.request.contextPath}/users/mypage">
        <span>마이페이지</span>
    </a>
</nav>
```

## 백엔드 컨트롤러와 뷰의 컨트롤러
jsp를 세팅하는 시점에서 이미 REST 기반의 백엔드 API는 완성되어 있었다.
```kotlin
@RestController  
@RequestMapping("/api/quiz/record")  
class QuizRecordController(  
    private val quizRecordService: QuizRecordServiceInterface  
) {  
  
    @PostMapping  
    fun createQuizRecord(@Valid @RequestBody request: QuizRecordRequest): DataResponse<Map<String, Long>> {  
        val recordId = quizRecordService.createQuizRecord(request)  
        return DataResponse.of(mapOf("recordId" to recordId))  
    }
}
```
해당 컨트롤러의 역할은 화면 이동과 Model 구성에 집중해야 하기 때문에
View를 위한 별개의 컨트롤러를 만들어 책임을 나누어 관리하도록 설계 했다.

## 존재하지 않는 경로에 대한 노출
위와 같이 존재하지 않는 url로 요청할 경우 다음과 같이 검은 화면이 나오기 시작했다.
![[스크린샷 2025-11-15 205854.png]]

**원인**
Spring MVC의 [[[Spring] DispatcherServlet|DispatcherServlet]] 단계에서 바로 잡히기 때문에, JSP 뷰가 로드되기 전에 
서버의 **기본 에러 핸들러**가 작동하여 JSON을 반환하는 것이다.

> 디스패처 서블릿은 HTTP 프로토콜로 들어오는 모든 요청을 가장 먼저 받아 
> 적합한 컨트롤러에 위임해주는 프론트 컨트롤러

``` kotlin
/**
 * 전역 예외 처리를 위한 컨트롤러 어드바이스
 * 모든 컨트롤러에서 발생하는 예외를 일관된 형태로 처리
 */
@RestControllerAdvice
class GlobalExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException::class, produces = [MediaType.APPLICATION_JSON_VALUE])
    fun handleValidationException(
        ex: MethodArgumentNotValidException
    ): ResponseEntity<ErrorResponse> {
        val errors = ex.bindingResult.fieldErrors.map { fieldError ->
            ErrorDetailResponse(
                field = fieldError.field,
                value = fieldError.rejectedValue,
                reason = fieldError.defaultMessage ?: "검증 실패"
            )
        }

        val errorResponse = ErrorResponse.of(ResponseCode.VALIDATION_FAILED, errors)
        return ResponseEntity.status(ResponseCode.VALIDATION_FAILED.httpStatus).body(errorResponse)
    }
    
    // 생략
}
```
기존에 전역 예외 처리를 관리하는 컨트롤러 어드바이저가 존재했다.

## 커스텀 에러 페이지
```kotlin
@ControllerAdvice  
class GlobalViewControllerAdvice {  
  
    /**  
     * 특정 HTTP 상태 코드를 반환하는 예외를 처리합니다.     * (예: EntityNotFoundException -> ResponseCode.NOT_FOUND로 매핑되는 경우)     */    @ExceptionHandler(Exception::class, produces = [MediaType.TEXT_HTML_VALUE])  
    fun handleGeneralViewException(  
        ex: Exception,  
        request: HttpServletRequest  
    ): String {  
        val status = if (ex is IllegalAccessException) {  
            HttpStatus.FORBIDDEN  
        } else {  
            HttpStatus.INTERNAL_SERVER_ERROR // 500  
        }  
  
        request.setAttribute("statusCode", status.value())  
        request.setAttribute("errorMessage", status.reasonPhrase)  
        request.setAttribute("exception", ex.message)  
  
        return "content/error/error"  
    }  
  
    @ExceptionHandler(NoHandlerFoundException::class, produces = [MediaType.TEXT_HTML_VALUE])  
    fun handleNotFoundError(  
        request: HttpServletRequest  
    ): String {  
        request.setAttribute("statusCode", HttpStatus.NOT_FOUND.value())  
        request.setAttribute("errorMessage", HttpStatus.NOT_FOUND.reasonPhrase)  
        request.setAttribute("exception", "요청하신 페이지를 찾을 수 없습니다.")  
  
        return "content/error/404"  
    }  
}
```
뷰 컨트롤러에 대한 어드바이저와 에러 페이지를 추가하여 예기치 못한 상황시 사용자가 대응할 수 있도록 수정했다.

그럼에도 불구하고 여전히 존재하지 않은 페이지를 입력시에 여전히 검은 화면을 제공했다.
```kotlin
/**
 * 기타 모든 예외에 대한 기본 처리
 */
@ExceptionHandler(Exception::class)
fun handleGenericException(
	ex: Exception
): ResponseEntity<ErrorResponse> {
	val errorResponse = ErrorResponse.of(ResponseCode.INTERNAL_ERROR, ex)
	return ResponseEntity.status(ResponseCode.INTERNAL_ERROR.httpStatus).body(errorResponse)
}
```
그 이유는 Rest controller의 어드바이저 내부에 이외의 모든 예외에 대한 기본 처리를 수행하는 핸들러가 존재했기 때문이다.

왜 하필이면 해당 에러 핸들러가 먼저 발생할까?
그 이유는 어드바이스가 **가장 잘 맞는 핸들러를 매칭**시켜주는 것이 아닌
단지 순서대로 순회 하다가 **가장 먼저 현재 예외에 매칭**되는 핸들러를 선택하기 때문이다.

> 여러 `ControllerAdvice`가 있을 때 `@Order`어노테이션으로 순서를 지정하지 않는다면 Spring은 `ControllerAdvice`를 임의의 순서로 호출합니다. 즉, 사용자가 예상하지 못한 예외 처리가 발생할 수 있습니다.

![[Pasted image 20251120134749.png]]
해당 핸들러를 삭제하니 최종적으로 다음과 같은 커스텀 에러 화면을 제공하는 것을 확인했다.
## Model과 DTO
```kotlin
fun showQuizRecordDetail(@PathVariable recordId: Long, model: Model, session: HttpSession): String {  
    val redirectPath = loginInterceptor(session)  
    if (redirectPath != null) return redirectPath  
  
    val response = quizRecordService.getUserQuizRecordDetail(recordId)  
  
    model.addAttribute("path", "content/quiz/record/detail.jsp")  
    model.addAttribute("data", response)  
  
    return "index"  
}
```
컨트롤러 내부에서는 기존에 만들었던 서비스와 연결하여 응답값 전체를 `data` 속성에 담아 전달한다.
# 출처
## jsp
[SpringBoot + jsp + intellij 사용을 위한 설정](https://milenote.tistory.com/176)
[코틀린과 jsp](https://devlemon.tistory.com/9)

## 어드바이저
[테코블-# ExceptionHandler와 ControllerAdvice를 알아보자](https://tecoble.techcourse.co.kr/post/2023-05-03-ExceptionHandler-ControllerAdvice/)
