# 정적 프로퍼티와 메서드
자바에서는 `static`이지만 코틀린에서는 `companion object` 내부에서 선언하여 구현

[Kotlin 기본 문법 8 : 정적 변수와 정적 메소드(feat. companion object)](https://colabear754.tistory.com/63)
```kotlin
// static == companion object  
class Calculator {  
    companion object {  
        var code = 123  
        fun sumStringList(strList: List<String>): Int {  
            return strList.sumOf { it.toInt() }  
        }  
    }  
}  
  
fun main() {  
    println(Calculator.code)  
    val sum = Calculator.sumStringList(listOf("1", "2", "3"))  
    println(sum)  
}
```
## 싱글톤
심지어 싱글톤을 자체적으로 지원함
```kotlin
fun main() {  
    println(Singleton.NAME)  
    println(Singleton.MIND)  
}  
  
// object는 자체적으로 지원하는 싱글톤이 적용된 클래스  
object Singleton {  
    val NAME = "유어진"  
    val MIND = "재밌게 살자"  
}
```
