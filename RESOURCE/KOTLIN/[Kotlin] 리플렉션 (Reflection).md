## 의존성 추가
### kotlin
```kotlin
dependencies { implementation(kotlin("reflect")) }
```
### groovy
```groovy
dependencies { implementation "org.jetbrains.kotlin:kotlin-reflect:2.2.21" }
```
### maven
```maven
<dependencies>
    <dependency>
        <groupId>org.jetbrains.kotlin</groupId>
        <artifactId>kotlin-reflect</artifactId>
    </dependency>
</dependencies>
```
### 직접 추가
[maven 레포지토리](https://mvnrepository.com/artifact/org.jetbrains.kotlin/kotlin-reflect/2.2.20)

## 리플렉션이란?
**런타임에 프로그램의 클래스를 조사하기 위해서 사용되는 기술**
- 프로그램이 실행할때 인스턴스를 통해 객체 내부 구조를 파악
- Spring에서 적극적으로 사용
## 리플렉션 접근
레퍼런스 객체를 만드는 방식은 2가지 방법이 있다
> 레퍼런스 객체 : 코틀린에서 리플랙션을 사용하기 위해 사용하는 객체
### 인스턴스
```kotlin
val i = ClassReference()  
val ik = i::class
```
### 객체
```kotlin
val k = ClassReference::class
```
## 리플렉션을 이용한 객체 생성
### createInstance()
```kotlin
val i = ClassReference()  
val ik = i::class
// 리플렉션으로 객체 생성  
val i2 = ik.createInstance()
```
![[Pasted image 20251119121542.png]]
### call() : 파라미터가 필요할때
```kotlin
class Person(val name: String) {  
    override fun toString(): String {  
        return "안녕하세요 저는 ${name}입니다!"  
    }  
}

// 

// 파라미터가 필요할 경우  
val personClass = Person::class  
val personPrConstructor = personClass.primaryConstructor  
val personInstance = personPrConstructor?.call("유어진")  
println(personInstance)
// result : 안녕하세요 저는 유어진입니다!
```

## 리플렉션을 이용한 변수 접근 및 수정
### 예시 수정
`year`과 `old()`메서드를 추가
```kotlin
class Person(val name: String) {  
    var age: Int = 0  
  
    private fun old(year: Int = 1): Int {  
        age++  
        return age  
    }  
  
    override fun toString(): String {  
        return "안녕하세요 저는 ${name}입니다! 나이는 ${age}입니다."  
    }  
}
```

### Class.functions
```kotlin
// 리플렉션을 이용한 함수 실행  
val oldFunction: KFunction<*>? = personClass.functions.find { it.name == "old" }  
// 해당 메서드가 private라면 Accessible = true를 통해 사용  
oldFunction?.isAccessible = true  
// 함수를, 실행함, 어디에?  
println("old() = ${oldFunction?.call(personInstance, 1)}")  
println(personInstance)
```
- `functions`로 해당 클래스가 갖고 있는 함수들을 접근
- `find`로 원하는 함수를 찾음
- `call`메서드를 사용하여 실행
### default 인자에 대한 주의점
```kotlin
private fun old(year: Int = 1): Int {  
	age++  
	return age  
}

println("old() = ${oldFunction?.call(personInstance)}")
```
`old`메서드의 인자는 default값이 있음에도 불구하고 이와 같이 호출하면 아래와 같이에러가 발생한다.
![[Pasted image 20251119124323.png]]
코틀린 리플렉션에서 `call()`은 모든 파라미터에 대해 값이 다 넘어온다는 가정하에 동작함
default가 있더라도 `call()`이 값을 채워주지는 않음
### callBy 활용
```kotlin
val params = oldFunction?.parameters  
val instanceParam = params?.first { it.kind.name == "INSTANCE" }  
  
oldFunction?.callBy(  
    mapOf(  
        instanceParam to personInstance  
    ) as Map<KParameter, Any?>  
)  
  
println(personInstance)
```
이럴땐 `callBy()`를 활용할 수 있다.
그러나 이는 오히려 더 어렵게 문제를 해결하는 방식이라고 생각한다.
이럴땐 매개변수에 값을 전달하는 것이 좋을듯하다.
## 리플렉션을 이용한 변수 접근 및 수정
`memberProperties`를 통하여 가져올 수 있음
```kotlin
// 리플렉션을 이용한 변수 접근 및 수정  
val props = personClass.memberProperties  
val ageProps = props.find { it.name == "age" }  
ageProps?.isAccessible = true  
ageProps?.get(personInstance!!) // get()은 nullable한 값을 받을 수 없기에 강제 선언
```

## 출처
[블로그1](https://sabarada.tistory.com/190)
[공식문서](https://kotlinlang.org/docs/reflection.html#class-references)
## 실습 코드
[github repository](https://github.com/yourjinKR/kotlinBasic/blob/e7996741f4f841beef2f550059544ba7a6e144a9/src/reflection/ClassReference.kt)
