# 고차함수
다른 함수를 인자로 받거나 함수를 반환하는 함수
코틀린에서는 람다나 함수 참조를 사용해 함수를 값으로 표현 가능

## 함수 타입
![Pasted image 20251119163902](Pasted%20image%2020251119163902.png)
코틀린은 타입 추론이 지원되기에 변수 타입을 지정하지 않고 람다 자체를 변수에 대입 할 수 있음

```kotlin
val sum = { a: Int, b: Int -> a + b }  
println(sum(1,2))
// result : 3
  
val sum2: (Int, Int) -> Int = { x, y -> x + y }  
println(sum2(1,2))
// result : 3
```
### null 처리
```kotlin
var canReturnNull: (Int, Int) -> Int? = {x, y -> null} // 결과값이 null일 수 있음
var funOrNull: ((Int, Int) -> Int)? = Null // 함수 자체가 null
```

## 함수 타입을 인자로 사용
![Pasted image 20251119165830](Pasted%20image%2020251119165830.png)

```kotlin
fun sumToString(num1: Int, num2: Int , function: (Int, Int) -> Int) {  
    val result = function(num1, num2)  
    println("${num1}과 ${num2}의 합은 $result 입니다.")  
}  
  
sumToString(1,2,sum)
// result : 1과 2의 합은 3 입니다.
```

## Default값을 활용
```kotlin
// 기본값 설정 가능  
fun String.sayName(func: (String) -> String = { "${this}!!" }) {  
    println(func(this))  
}  
"유어진".sayName { "야!! $it!!" }  
"유어진".sayName()
// 야!! 유어진!!
// 유어진!!
```

## 함수를 함수에서 반환
```kotlin
// 함수를 함수에서 반환  
fun makePlusWithStart(start: Int): (Int, Int) -> Int {  
    return { a, b -> start + a + b }  
}  
  
val sumWithTwo = makePlusWithStart(2)  
val sumResult = sumWithTwo(1,2)  
println(sumResult)
// 5
```

## 중복제거
코틀린 람다의 가장 최대 장점이라고 할 수 있다.
```kotlin
fun repeatUntilSuccess(action: () -> Unit) {  
    while (true) {  
        try {  
            action()  
            return  
        } catch (e: IllegalArgumentException) {  
            println(e.message)  
        }  
    }  
}
```
위 코드는 인자에 함수를 받고 함수 내부 행위를 예외가 발생하지 않을 때까지 반복해주는 함수이다.

```kotlin
repeatUntilSuccess {  
    val purchaseRequest = inputView.purchaseAmount()  
    val purchaseResponse = lottoService.purchaseLotto(purchaseRequest, lottoGame)  
    outputView.purchaseResult(purchaseResponse)  
}  
  
repeatUntilSuccess {  
    val winningRequest = inputView.winningLotto()  
    val winningResponse = lottoService.getWinningResult(winningRequest, lottoGame)  
    outputView.winningResult(winningResponse)
}
```
위와 같이 사용하여 쉽게 적용 가능하다

# 출처
[블로그1](https://velog.io/@akimcse/Kotlin-in-Action-8.-%EA%B3%A0%EC%B0%A8-%ED%95%A8%EC%88%98-%ED%8C%8C%EB%9D%BC%EB%AF%B8%ED%84%B0%EC%99%80-%EB%B0%98%ED%99%98-%EA%B0%92%EC%9C%BC%EB%A1%9C-%EB%9E%8C%EB%8B%A4-%EC%82%AC%EC%9A%A9)