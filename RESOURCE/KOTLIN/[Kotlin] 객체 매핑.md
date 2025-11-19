
## 확장함수 활용
```kotlin
import org.springframework.ui.Model

fun Model.addAll(map: Map<String, Any?>) {
    map.forEach { (key, value) ->
        if (value != null) {
            this.addAttribute(key, value)
        }
    }
}
```
### 사용
```kotlin
val map = mapOf(  
    "path" to "content/quiz/config/detail.jsp",  
    "configId" to response.configId,  
    "quizName" to response.quizName,  
    "level" to response.level,  
    "updatedAt" to response.updatedAt,  
    "createdAt" to response.createdAt,  
    "quizzes" to response.quizzes,  
)  
  
model.addAll(map)
```

## 리플렉션과 확장함수 활용
```kotlin
inline fun <reified T : Any> Model.addFrom(dto: T) {  
    T::class.memberProperties.forEach { prop ->  
        val value = prop.get(dto)  
        this.addAttribute(prop.name, value)  
    }  
}
```

```kotlin
model.addFrom(response)
```
해당 방식을 사용한다면 함수 하나로 이와 같이 구현 가능
