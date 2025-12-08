# 확장함수 (Function Extension)
- 클래스에 직접 추가하는 것과 같이 사용되지만 실질적으로는 클래스 밖에서 선언
- 주로 남이 만든(라이브러리) 클래스에 기능을 추가하고 싶을때 사용
## 개념
- 어떤 클래스의 멤버 메서드인 것처럼 호출할 수 있지만 그 클래스의 밖에서 선언된 함수
- 따로 상속받지 않고 하나의 클래스에 추가적인 메서드를 구현하고 싶을 때 사용하는 함수
- 새로운 클래스를 만드는 번거로움을 줄일 수 있음
- 확장 함수는 각 클래스에 **정적 메서드**로 생성되기 때문에 virtual method invocation이 발생하지 않음
## 특징
- 오버라이딩 불가
- 오버로드 가능
- 멤버 메서드명과 확장 함수명이 같을땐 멤버 메서드가 우선적 실행
- `import`를 통해 다른 파일에서도 사용 가능

```kotlin
class User(  
    val name: String,  
    var age: Int,  
    var intro: String = "안녕하세요 저는 $name 입니다!")
}
```

```kotlin
// 확장 함수  
fun User.coding() {  
    println("코딩을 함")  
}  
  
// this로 멤버 변수 접근 가능  
fun User.coding(lang: String) {  
    println("${this.name}이(가) $lang 코딩을 함")  
}
```
이처럼 클래스에 메서드를 추가하고 싶을때 손쉽게 사용 가능

## 활용 예시
### 문자열의 마지막 글자 가져오기 
```kotlin
fun String.getLastChar(): Char {  
    return this[length-1]  
}  
```
### 객체 매핑
[Kotlin-객체-매핑](Kotlin-객체-매핑.md)

# 확장 프로퍼티 (Extension Property)
## 활용 예시
### 마지막 글자 가져오기
```kotlin
var StringBuilder.lastChar: Char
    get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
    }
    
fun main() {
    val sb = StringBuilder("Gold")
    sb.lastChar = 'f'
    println(sb) // golf
}
```

## 정적 바인딩
```kotlin
open class Idol {
    open fun sing() = println("아이돌이 노래를 불러요")
}
 
class BTS: Idol() {
    override fun sing() = println("BTS가 노래를 불러요")
}
 
fun Idol.dance() = println("아이돌이 춤을 춰요")
fun BTS.dance() = println("BTS가 춤을 춰요")
    
fun main() {
    // 부모 타입으로 자식 객체 생성
    val bts: Idol = BTS()
    bts.sing() // BTS가 노래를 불러요
    bts.dance() // 아이돌이 춤을 춰요
}
```
일반 함수는 동적 바인딩 되어서 -> 런타임 시간에 함수의 메모리 주소가 결정되는 거고
**확장 함수**는 **정적 바인딩** 되어서 -> 컴파일 시간에 Idol의 메모리 주소로 결정되어 버린다.

# 출처
[공식 문서](https://kotlinlang.org/docs/extensions.html#generic-extension-functions)
