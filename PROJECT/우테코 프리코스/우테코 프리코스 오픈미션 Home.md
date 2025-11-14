# 무료 배포
https://aamoos.tistory.com/756#google_vignette

# 코틀린이 뭘까?
[[10분 테코톡] 부나의 Java에서 Kotlin으로](https://www.youtube.com/watch?v=eA8e18ddSms)
## 탄생 과정
기존의 자바 : 자바는 유지보수에 골칫거리가 많음, 단순한 기능에도 너무 많은 자바 코드가 필요함

아래와 같은 목적을 지니면서 Kotlin이 생겨났다.
1. 속도
2. 상호 운용성
3. 간결성

## Kotilin 장점
### 1. 정적 타입 지정 언어
Kotlin은 정적 타입 지정 언어이며 컴파일러가 코드를 검증하고 컴파일하기 때문에 런타임 오류를 줄임

### 2. 타입 추론
변수 선언 시 타입을 작성하지 않아도 `val`, `var` 키워드만으로도 가능하다.
```kotlin
val name = "유어진"
var age = 26
val mind = "오픈 미션 화이팅 !!!"
```
- `val` : final
- `var` : 가변 

코틀린에서는 필드가 아닌 프로퍼티라고 말한다. 그 이유는 getter와 setter를 자동으로 만든다.

### 3.  Null Safety
Kotlin은 Null을 다루는 방식이 명확하게 정해져 있기 때문에 Null Saftey를 보장한다.
```kotlin
fun main() {
    var name: String? = null // ?를 붙이면 Nullable함
    println(name.length) // 여기서부터 경고 발생
}
```

```kotlin
fun main() {
    var name: String? = null 
    println(name?.length) // ?를 붙여 해결 가능
}
```
이와 같은 과정은 당연히 실행하기 전부터 컴파일러가 코드를 검증하여 알려줌 

**Evis 연산자 [ ?: ]**
```java
fun main() {
    var name: String? = null
    println(name ?: "yourjin")
}
```

### 4. 코드의 간결성과 명확성
Kotlin은 Java에 비해 코드를 간결하고 명확하게 작성할 수 있다.
그렇기 때문에 비교적 코드를 파악하기 쉽다는 장점이 있다.

**SAM(Single Abstract Method)**
단 하나의 추상화 함수를 가지고 있는 함수형 인터페이스를 뜻한다.

### 5. Coroutine
비동기적 코드를 쉽게 작성할 수 있도록 해당 기능을 제공
> 코틀린 고유의 개념은 아님!

## Java와 상호 운용
코틀린 코드를 컴파일시 자바 바이트 코드를 만들기 때문에 똑같이 동작

# 안드로이드 스튜디오
[안드로이드-코틀린으로-하는-안드로이드-앱-프로그래밍-1](https://velog.io/@jihyeon9975/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EC%BD%94%ED%8B%80%EB%A6%B0%EC%9C%BC%EB%A1%9C-%ED%95%98%EB%8A%94-%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EC%95%B1-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-1-)

# 참고 서적
동네 도서관에 있는 관련서적들을 찾음,
그러나 짧은 기간 앱 출시라는 무모한 도전을 펼쳤으니 이에 맞는 서적을 찾아봄.

[이것이 안드로이드다 : with 코틀린 : 안드로이드 입문의 3가지 장벽, 언어+실전+환경 완벽 대응!](https://www.snlib.go.kr/hor/menu/11085/program/30009/plusSearchResultDetail.do?searchType=SIMPLE&searchMenuCollectionCategory=&searchMenuEBookCategory=&searchCategory=BOOK&searchKey=ALL&searchKey1=&searchKey2=&searchKey3=&searchKey4=&searchKey5=&searchKeyword=%EC%BD%94%ED%8B%80%EB%A6%B0&searchKeyword1=&searchKeyword2=&searchKeyword3=&searchKeyword4=&searchKeyword5=&searchOperator1=&searchOperator2=&searchOperator3=&searchOperator4=&searchOperator5=&searchPublishStartYear=&searchPublishEndYear=&searchLibrary=&searchPbLibrary=&searchSmLibrary=&searchLibraryArr=MH&searchRoom=&searchKdc=&searchSort=SIMILAR&searchOrder=DESC&searchRecordCount=10&searchBookClass=ALL&currentPageNo=1&viewStatus=IMAGE&preSearchKey=ALL&preSearchKeyword=%EC%BD%94%ED%8B%80%EB%A6%B0&reSearchYn=N&recKey=1849620492&bookKey=1849620494&publishFormCode=BO)

