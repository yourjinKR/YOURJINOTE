# 람다란?
기본적으로 다른 함수에 넘길 수 있는 작은 코드 조각을 의미

## 고차함수
기본적으로 코틀린의 함수는 일급 객체이다.
간단히 일급 객체라는 것은 함수를 변수나 자료구조로 담아낼 수 있고, 
파라미터로 다른 함수로 전달하거나 함수를 리턴 가능하는 것을 의미한다.
[[Kotlin-High-order-Function|view more...]]

# 람다 사용 이유
## 간결함
가독성이 좋은 코드를 지향하기 위해 만들어짐
간단한 구현을 위해서 익명 클래스를 이용하는건 과함
# 람다 표기법
```kotlin
// 매개변수 없음
() -> 3

// 매개변수 있음
(a,b) -> a+b

// 생략 전
(a: Int, b: Int) -> Int -- O

// 생략 후
(a, b) -> Int -- O

// 생략 불가능
(a, b) ->  -- X
```

# 출처
[공식문서](https://kotlinlang.org/docs/lambdas.html)
[블로그1](https://velog.io/@betalabs/Kotlin-%EB%9E%8C%EB%8B%A4-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0)

