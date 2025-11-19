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

## 존재하지 않는 경로에 대한 노출
위와 같이 존재하지 않는 url로 요청할 경우 다음과 같이 검은 화면을 제공

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
해당 핸들러는 위에서 지정한 예외 상황을 제외한 모든 예외에 대한 기본 처리를 수행함.
지금 상황은 이 핸들러가 작동된 것이다.



## 백엔드 컨트롤러와 뷰의 컨트롤러


## Model과 DTO
