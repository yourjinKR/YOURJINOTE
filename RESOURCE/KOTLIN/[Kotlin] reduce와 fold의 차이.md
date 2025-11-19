둘 다 컬렉션 내 데이터를 모으는 역할을 수행한다.
둘의 차이점은 초기값이 있는가 없는가의 차이이다

```kotlin
val list = listOf(1,2,3,4,5)  
println(list.reduce { acc, i -> acc + i }) // 15
println(list.fold(100) { acc, i -> acc + i }) // 115
```

```kotlin
val emptyList = emptyList<Int>()  
println(emptyList.reduce { acc, i -> acc + i }) // UnsupportedOperationException  
println(emptyList.fold(100) { acc, i -> acc + i })
```
추가로 `fold`는 null-safe하다.
빈 배열과 같이 요소에 접근시 null일 경우 `reduce`는 `UnsupportedOperationException`가 발생한다.

## fold 활용 예시
### 객체 세기
```kotlin
data class UserCount(val total: Int, val admins: Int)

val users = listOf(
    User("A", role = "ADMIN"),
    User("B", role = "USER"),
    User("C", role = "ADMIN")
)

val count = users.fold(UserCount(0, 0)) { acc, user ->
    acc.copy(
        total = acc.total + 1,
        admins = acc.admins + if (user.role == "ADMIN") 1 else 0
    )
}
```
### 카테고리 카운트
```kotlin
val logs = listOf("INFO", "ERROR", "INFO", "WARN", "ERROR")

val result = logs.fold(mutableMapOf<String, Int>()) { acc, log ->
    acc[log] = (acc[log] ?: 0) + 1
    acc
}

println(result)
// {INFO=2, ERROR=2, WARN=1}
```