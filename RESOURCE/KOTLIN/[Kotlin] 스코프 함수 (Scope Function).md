[[[Kotlin] 확장 함수]]
# 스코프 함수란?
어떠한 객체에 대하여 해당 객체가 다루는 특정 범위를 생성하여 프로퍼티나 메소드를 처리하는 코드를 간결하게 만들거나, 메소드 체이닝에 활용하는 함수를 의미한다. 

# 종류
- let
- run
- also
- apply
- with

# let
- 원본 객체를 파라미터로 받는 일반 함수
- 함수의 동작을 수행한 후 반환
- 파라미터 접근을 위해서는 `it`을 사용해야 하나 이름 변경 가능
```kotlin
// let  
val person = Person("유어진", 20, "성남").let {  
    println(it)  
    it.moveTo("서울")  
    it.incrementAge()  
    it  
}  
println(person)

// 이름 변경
val person = Person("유어진", 20, "성남").let { p ->  
    println()  
    p.moveTo("서울")  
    p.incrementAge()  
    p  
}
```
# run
- 파라미터로 받는 함수가 원본 객체에 대한 확장 함수
- `this` 키워드를 통해 접근하며 이름은 수정 불가능하나 생략은 가능
```kotlin
// run 1  
val person2 = Person("박명수", 20, "성남").run {  
    println(this)  
    this.moveTo("서울")  
    this.incrementAge()  
    this  
}  
println(person2)  

// run 2  
val result = run {  
    val person = Person("정준하", 20, "성남")  
    println(person)  
    person.moveTo("서울")  
    person.incrementAge()  
    person  
}  
println(result)
```
- 추가로 아무런 파라미터를 받지 않고 실행할 경우 블럭 내부의 코드를 실행 후 결과를 저장한다
# also
- 파라미터로 받는 bolck이라는 함수가 원본 객체를 파라미터로 받는 일반 함수
- 내부 동작 수행 후 원본 객체를 반환
```kotlin
// also 1  
val alsoResult = Person("유재석", 20, "성남").also {  
    println(it)  
    it.moveTo("서울")  
    it.incrementAge()  
}  
println(alsoResult)
```
### 활용 : 입력값이 null일때까지 받기 
```kotlin
var str: String? = ""
while (readLine().also { str = it }?.isEmpty() == false) {
    ...
}
```
### 활용 : 값 바꾸기
```kotlin
fun main() {
    var a = 10
    var b = 20
    println("a = $a, b = $b")
    b = a.also { a = b }
    println("a = $a, b = $b")
}
 
```
# apply
- 파라미터로 받는 함수가 원본 객체의 확장 함수
- block 내부의 동작을 수행한 후 원본 객체를 반환
- 원본 객체의 멤버에 접근하기 위해서는 `this` 사용, 변경 X, 생략 가능
```kotlin
val applyResult = Person("하하", 20, "성남").apply {  
    println(this)  
    moveTo("서울")  
    incrementAge()  
}  
println(applyResult)
```
# with
- 확장함수가 아닌 일반 함수
- 객체를 파라미터로 받는 일반 함수이기에 반드시 receiver는 nom-null이다.
- 그렇기에 non-null인 객체의 프로퍼티나 메소드를 호출하는 코드를 묶을때 사용

# 사용 이유
또한 스코프 함수들은 파라미터로 받은 값이 null일 경우 실행하지 않음.
그렇기에 null-safe 작업에 이점이 있다.
```kotlin
fun updateProfile(password: String?, username: String?, level: Level?) {  
    if (password != null) {  
        this.password = password  
    }  
    if (username != null) {  
        this.username = username  
    }  
    if (level != null) {  
        this.level = level  
    }  
}
```

```kotlin
fun updateProfile(password: String?, username: String?, level: Level?) {  
    password?.let { this.password = it }  
    username?.let { this.username = it }  
    level?.let { this.level = it }  
}
```

# 정리
|함수|람다에서 객체 이름|반환값|주 사용 목적|
|---|---|---|---|
|`let`|`it`|람다 결과|**null-safe, 변환(map), 임시 스코프**|
|`run`|`this`|람다 결과|객체 기반 계산, 초기화|
|`with`|`this`|람다 결과|객체 외부에서 사용하기 좋은 run|
|`apply`|`this`|객체 자신|**객체 설정(빌더 패턴)**|
|`also`|`it`|객체 자신|**side-effect 처리(로그 등)**|
# 출처
https://colabear754.tistory.com/80