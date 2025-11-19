해당 이슈는 Kotlin + Spring + JSP 프로젝트를 마친 후 리팩토링 과정에서 발생한 일들을 작성했습니다 !

## 기존의 객체 매핑 방식
Spring FrameWork에서는 `Model`이라는 객체로 
간단하게 데이터를 송수신 할 수 있다.

```kotlin
model.addAttribute("data", response)
```
기존에는 다음과 같이 응답값 자체를 그대로 넘기는 식으로 진행했다.

```jsp
<div class="summary-card">
	<h2>${data.quizName}</h2>

	<div class="score-display">
		<span class="score-value">${data.score}</span>
		<span class="score-total">/ ${fn:length(data.answers)}</span>
	</div>

	<div class="summary-meta">
		<span class="level-badge">${data.level}</span>
		<span>
			완료: <c:out value="${data.completedAt}"/>
		</span>
	</div>
</div>
```
그로 인해 이와 같은 방식으로 프로퍼티를 접근했다.

## 앞에 data를 없애보자
```jsp
<c:forEach var="quizName" items="${data.quizName}" varStatus="status">
<c:forEach var="score" items="${data.score}" varStatus="status">
```
이와 같은 방식으로 값을 지정하여 관리해야 될 것이다.

```kotlin
model.addAttribute("path", "content/quiz/config/detail.jsp")  
model.addAttribute("configId", response.configId)  
model.addAttribute("quizName", response.quizName)  
model.addAttribute("level", response.level)  
model.addAttribute("updatedAt", response.updatedAt)  
model.addAttribute("createdAt", response.createdAt)  
model.addAttribute("quizzes", response.quizzes)
```
혹은 서버에서 이와 같이 각각의 프로퍼티를 대입해야 한다.
좀 더 쉽게 대입하는 방법은 없을까?

```kotlin
val path = "content/quiz/config/detail.jsp"  
model.addAllAttributes(  
    listOf<Any>(  
        path,  
        response.configId,  
        response.quizName,  
        response.level,  
        response.updatedAt,  
        response.createdAt,  
        response.quizzes,  
    )
)
```
다행이 `addAllAttributes`가 있어 이를 활용해봤다.
그러나 에러 발생한다.

```java
for (value in list) {
    model.addAttribute(value) // 이름 없이
}
```
해당 메서드는 내부적으로 이와 같이 동작하며
이름 없이 attribute를 add하기에 타입이 이름이 되버린다

```jsp
${string}   <!-- path, quizName 중 어느 쪽이 들어가 있을지 모름 -->
${long}     <!-- configId -->
```

```kotlin
model.addAllAttributes(  
    mapOf(  
        "path" to "content/quiz/config/detail.jsp",  
        "configId" to response.configId,  
        "quizName" to response.quizName,  
        "level" to response.level,  
        "updatedAt" to response.updatedAt,  
        "createdAt" to response.createdAt,  
        "quizzes" to response.quizzes,  
    )  
)
```
그렇기에 이런식으로 만들어야 한다.

이러한 반복적인 코드는 개발자를 괴롭게 할 뿐만 아니라 실수하기 딱 좋은 코드이다.

### 팀원이 작성한 작성한 코드 중...
```kotlin
model.addAttribute(  
    "data", mapOf(  
        "users" to users,  
        "currentUserId" to currentUser.id  
    )  
)
```
이렇게까지 매핑하여 구현한 팀원들은 
요구사항 변경 소식을 듣게 된다면 매우 분노할 것이다.
그러면 어떻게 해야할까?

이를 쉽게 매핑할 수는 없을까?

## 과정
### 이렇게 하면 될까?
자바의 리플렉션과 같이 코틀린에서도 리플렉션을 지원함을 알게 되었고 이에 대해 가능성을 보았다.

```kotlin
fun <T> Model.addFrom(dto: T) {  
    val props = dto!!::class.memberProperties  
    props.forEach {  
        val value = it.get(dto)  
        this.addAttribute(it.name, value)  
    }  
}
```
![[Pasted image 20251119184825.png]]
get()이 요구하는 리시버의 타입이 맞지 않았기에 아래와 같이 추가 수정했다.

```kotlin
fun Model.addFrom(dto: Any) {  
    val props = dto::class.memberProperties  
    props.forEach {  
        val value = (props as KProperty1<Any, *>).get(dto)  
        this.addAttribute(it.name, value)  
    }  
}
```
해당 코드 또한 `ClassCastException`이 발생했다.
형변환 과정에서 예외가 발생한 것이다.

```
2025-11-19T18:54:25.737+09:00  WARN 27392 --- [wordCapsule] [nio-8080-exec-4] .m.m.a.ExceptionHandlerExceptionResolver : Resolved [java.lang.ClassCastException: class java.util.ArrayList cannot be cast to class kotlin.reflect.KProperty1 (java.util.ArrayList is in module java.base of loader 'bootstrap'; kotlin.reflect.KProperty1 is in unnamed module of loader 'app')]
2025-11-19T18:54:25.831+09:00  WARN 27392 --- [wordCapsule] [nio-8080-exec-7] .m.m.a.ExceptionHandlerExceptionResolver : Resolved [org.springframework.web.servlet.resource.NoResourceFoundException: No static resource favicon.ico.]
```

### 안전하게 필터링 하기
```kotlin
fun Model.addFromTest(dto: Any) {  
    dto::class.memberProperties  
        .filterIsInstance<KProperty1<Any, *>>()  
        .forEach { prop ->  
            val value = prop.get(dto)  
            this.addAttribute(prop.name, value)  
        }  
}
```
`filterIsInstance`를 사용하여 안전하게 인스턴스 확인 후 필터링을 수행했다.
[코틀린 다양한 filter 메서드 참고 블로그](https://blog.yena.io/studynote/2020/01/22/Kotlin-Collection-Filter.html)
[filterIsInstance 참고 블로그](https://velog.io/@windsekirun/Kotlin-filterIsInstance-with-reflection-NumberPicker-text-color)

## 최종
제네릭을 활용하여 
```kotlin
inline fun <reified T : Any> Model.addFrom(dto: T) {  
    T::class.memberProperties.forEach { prop ->  
        val value = prop.get(dto)  
        this.addAttribute(prop.name, value)  
    }  
}
```
### 사용
```kotlin
model.addFrom(response)
```

