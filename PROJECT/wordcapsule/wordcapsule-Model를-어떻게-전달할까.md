프로젝트 후 혼자 든 생각을 기반으로 리팩토링 시도

## 기존의 객체 매핑 방식
Spring FrameWork에서는 `Model`이라는 객체로 간단하게 데이터를 송수신 할 수 있다.

```kotlin
model.addAttribute("data", response)
```
기존에는 다음과 같이 응답값 자체 `data`라는 명칭으로 통일 시켜 그대로 넘기는 식으로 진행했다.

```html
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

그러나 속성에 접근할때 고정적으로 `data`를 붙여야 하는 상황에 대해 의문점을 가지기 시작했다.
> 반복작업은 너무 싫어~
## 앞에 data를 없애보자
```html
<c:forEach var="quizName" items="${data.quizName}" varStatus="status">
<c:forEach var="score" items="${data.score}" varStatus="status">
```
jsp에서 해결한다면 이와 같은 방식으로 값을 지정하여 관리해야 될 것이다.
> 귀찮은데?

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
> 귀찮은데?

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
> 이것도 귀찮은데?

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
이렇게까지 매핑하여 구현한 팀원들은 요구사항 변경 소식을 듣게 된다면 매우 분노할 것이다.
그러면 어떻게 해야할까? 이를 쉽게 매핑할 수는 없을까?

## 과정
### 이렇게 하면 될까?
자바의 리플렉션과 같이 코틀린에서도 리플렉션을 지원함을 알게 되었고 이에 대해 가능성을 보았다.

우선 아래와 같이 생각의 흐름대로 작성해봤다.
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
런타임 과정에서 제네릭의 타입은 소거되기에 이는 당연히 작동하지 않는 코드이다.
> 또 자바에 대한 기초 부족 이슈..

```kotlin
fun Model.addFrom(dto: Any) {  
    val props = dto::class.memberProperties  
    props.forEach {  
        val value = (props as KProperty1<Any, *>).get(dto)  
        this.addAttribute(it.name, value)  
    }  
}
```
해당 코드 또한 `ClassCastException`이 발생했다. 형변환 과정에서 예외가 발생한 것이다.

```
2025-11-19T18:54:25.737+09:00  WARN 27392 --- [wordCapsule] [nio-8080-exec-4] .m.m.a.ExceptionHandlerExceptionResolver : Resolved [java.lang.ClassCastException: class java.util.ArrayList cannot be cast to class kotlin.reflect.KProperty1 (java.util.ArrayList is in module java.base of loader 'bootstrap'; kotlin.reflect.KProperty1 is in unnamed module of loader 'app')]
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
그러나 여전히 `KProperty1<Any, *>`와 같은 캐스팅은 안정성에 문제가 있다고 판단했다.
### 다시 제네릭을 사용해보자
```kotlin
fun <T> Model.addFromTest2(dto: T) {  
    dto!!::class.memberProperties  
        .filterIsInstance<KProperty1<T, *>>()  
        .forEach { prop ->  
            val value = prop.get(dto)  
            this.addAttribute(prop.name, value)  
        }  
}
```
제네릭을 활용하여 필터링 과정을 조금 더 안정적으로 수행하도록 수정했다.
### filterIsInstance() 를 없애보자
`filterIsInstance`를 사용하여 구현하는 방식은 안되는 코드를 억지로 끼워 맞춘 경향이 있다.
이를 없애기 위해 리플렉션 객체를 만들때 인스턴스가 아닌 객체로 접근하는 방식으로 진행했다.
```kotlin
 fun <T> Model.addFromTest3(dto: T) {  
    T::class.memberProperties  
        .filterIsInstance<KProperty1<T, *>>()  
        .forEach { prop ->  
            val value = prop.get(dto)  
            this.addAttribute(prop.name, value)  
        }  
}
```
당연히 JVM 런타임 과정에서 Generic의 타입을 소거하기 때문에 `T::class`는 동작하지 않는다.

![[Pasted image 20251120192511.png]]
`reified`가 뭐지?

```kotlin
val value = listOf<String>("abcd", "efg")  
if (value is List<String>) { // 이 검사는 허용된다.
}
```
위 코드 처럼 코틀린 컴파일러는 컴파일 시점에 명시적으로 
타입을 알 수 있다면 타입 검사의 수행을 허용한다.

이러한 특징이 있기에 해당 함수를 `inline`하고 제네릭에 `reified`키워드를 붙인다면
컴파일 과정에서 함수가 inline으로 바뀌기에 타입이 명시된다.

```kotlin
inline fun <reified T : Any> Model.addFromTest3(dto: T) {  
    T::class.memberProperties  
        .filterIsInstance<KProperty1<T, *>>()  
        .forEach { prop ->  
            val value = prop.get(dto)  
            this.addAttribute(prop.name, value)  
        }  
}
```
그로 인해 리플렉션 객체 생성 지점은 해결이 됐으나, `memberProperties`에 아래와 같이 타입에 대한 경고가 발생했다.
![[Pasted image 20251120193427.png]]

```kotlin
inline fun <reified T : Any> Model.addFromTest3(dto: T) {  
    T::class.memberProperties  
        .toList()  
        .forEach { prop ->  
            val value = prop.get(dto)  
            this.addAttribute(prop.name, value)  
        }  
}
```
이후 제네릭에 `Any`를 추가하여 해결했다. 이후 `toList()`와 같은 반복작업은 삭제하였고 아래와 같은 코드가 완성됐다.
## 최종
```kotlin
/**  
 * DTO를 Model attribute에 자동 매핑 함수
*/
inline fun <reified T : Any> Model.addAllAttributesFrom(dto: T) {  
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

# 테스트
## 테스트용 DTO
```kotlin
data class TestDto(  
    val id: Long,  
    val name: String,  
    val detail: TestDetailDto,  
    val list: List<TestListDto>,  
    val isNull: Any?  
)  
  
data class TestDetailDto(  
    val id: Long,  
)  
  
data class TestListDto(  
    val id: Long  
)

private fun setUpDto(): TestDto {  
    return TestDto(  
        id = 1L,  
        name = "test",  
        detail = TestDetailDto(1L),  
        list = listOf(  
            TestListDto(1L),  
            TestListDto(2L),  
            TestListDto(3L),  
        ),  
        isNull = null,  
    )  
}
```
## 테스트 코드
```kotlin
private fun newModel(): Model = ExtendedModelMap()  
  
companion object {  
  
    @JvmStatic  
    fun addFromMethods(): Stream<Arguments> {  
        return Stream.of(  
            Arguments.of(  
                "addFromTest",  
                { model: Model, dto: TestDto ->  
                    model.addFromTest(dto)  
                }  
            ),  
            Arguments.of(  
                "addFromTest2",  
                { model: Model, dto: TestDto ->  
                    model.addFromTest2(dto)  
                }  
            ),  
            Arguments.of(  
                "addFromTest3",  
                { model: Model, dto: TestDto ->  
                    model.addFromTest3(dto)  
                }  
            ),  
            Arguments.of(  
                "addFromTest4",  
                { model: Model, dto: TestDto ->  
                    model.addAllAttributesFrom(dto)  
                }  
            ),  
        )  
    }  
}  
  
@ParameterizedTest(name = "{0} - 프로퍼티명과 값 매핑 테스트")  
@MethodSource("addFromMethods")  
@DisplayName("DTO 프로퍼티명과 값이 Model에 동일하게 매핑된다")  
fun testNameAndValue(  
    methodName: String,  
    mapper: (Model, TestDto) -> Unit  
) {  
    // given  
    val dto = setUpDto()  
    val model = newModel()  
  
    // when  
    mapper(model, dto)  
  
    // then  
    val asMap = model.asMap()  
  
    // 기본 타입  
    assertThat(asMap["id"])  
        .`as`("$methodName - id 프로퍼티 매핑")  
        .isEqualTo(1L)  
    assertThat(asMap["name"])  
        .`as`("$methodName - name 프로퍼티 매핑")  
        .isEqualTo("test")  
  
    // detail  
    assertThat(asMap["detail"])  
        .`as`("$methodName - detail 프로퍼티 타입")  
        .isInstanceOf(TestDetailDto::class.java)  
    val detail = asMap["detail"] as TestDetailDto  
    assertThat(detail.id)  
        .`as`("$methodName - detail.id 값")  
        .isEqualTo(1L)  
  
    // list  
    assertThat(asMap["list"])  
        .`as`("$methodName - list 프로퍼티 타입")  
        .isInstanceOf(List::class.java)  
    val list = asMap["list"] as List<*>  
    assertThat(list).hasSize(3)  
    val first = list[0] as TestListDto  
    val second = list[1] as TestListDto  
    val third = list[2] as TestListDto  
    assertThat(first.id).isEqualTo(1L)  
    assertThat(second.id).isEqualTo(2L)  
    assertThat(third.id).isEqualTo(3L)  
  
    // null check  
    assertThat(asMap["isNull"]).isNull()  
}
```

## 과연 괜찮은걸까?
그냥 모든 프로퍼티에 `data`가 붙는 것이 굉장히 거슬려서 시작한 일이다.
막상 구현을 마치고 보니 이와 같이 명시하는 것이 해당 프로퍼티가 Model임을 명확히 알게 될 수 있다고 생각이 든다.
## PR
이렇게 혼자 리팩토링 과정을 거쳤지만 근본적으로 괜찮은지에 대해서도 확신을 가지지 못했다.
이에 대해 팀원들은 어떨지 궁금하여 해당 작업을 PR에 개시했다.
막상 올리기 직전에 부끄러워서 이를 게시 할지말지 고민했지만 "에라 모르겠다" 심정으로 작성했다.
https://github.com/woowacourse-openmission/wordcapsule/pull/42

## 도움이 됐던 자료
[코틀린 다양한 filter 메서드 참고 블로그](https://blog.yena.io/studynote/2020/01/22/Kotlin-Collection-Filter.html)
[filterIsInstance 참고 블로그](https://velog.io/@windsekirun/Kotlin-filterIsInstance-with-reflection-NumberPicker-text-color)
[코틀린 테스트코드 참고 블로그](https://damon-911.tistory.com/entry/Kotlin-JUnit%EC%9C%BC%EB%A1%9C-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0)
[generic 타입 & reified 키워드 참고 글](https://jaeyeong951.medium.com/kotlin-generic-%ED%83%80%EC%9E%85-reified-1726e9a23d40)