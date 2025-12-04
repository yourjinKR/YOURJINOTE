클래스의 멤버변수를 자바에서는 **필드**, 코틀린에서는 **프로퍼티**라고 부른다.
## 필드와 프로퍼티
가장 큰 차이로는 프로퍼티는 **정적**인 반면에 필드는 **동적**이다.
### 프로퍼티
- 정적
- 객체가 갖는 상태와 같은 속성
	- 필드
	- 생성자
	- 메서드 (get, set)
### 필드
- 동적
- 프로퍼티의 실제 값

![Pasted image 20251118170601](Pasted%20image%2020251118170601.png)

## 자바의 필드
### 특징
- 데이터만 저장하는 변수 자체
- 캡슐화를 위해 getter/setter 작성 필수
- 필드 자체는 로직을 포함하지 않음
- Lombok 안 쓰면 boilerplate 코드가 많아짐
## 코틀린의 프로퍼티
### 특징
- 백킹 필드(field)
- 자동 생성된 getter
- 자동 생성된 setter
> Backing Field는 프로퍼티에 대한 접근을 감시하고, 제어하는 숨겨진 필드를 말한다.
## 왜 프로퍼티로 만들었나요?
- 코틀린 사용시 보일러플레이트를 줄일 수 있다.
- 더욱 안전한 캡슐화를 보장
### 자바
```java
public class User {
    private String name;
    private int age;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}
```
### 코틀린
```kotlin
class User(
    var name: String,
    var age: Int
)
```
## validation 사용시 차이점
코프링 프로젝트에서 Validator를 사용 중 IDE에서 다음과 경고를 보여줬다.

> validator : 아니,,, 뭘 검증하라구요 주인님?

![Pasted image 20251118164005](Pasted%20image%2020251118164005.png)
이는 해당 프로퍼티에 어떤 영역에서 검증을 거칠 것인가에 대해 명확하게 명시하라는 뜻이다.
코틀린에서 프로퍼티를 선언시 해당 프로퍼티가 수행하는 역할이 많기 때문이다
- 필드
- 생성자
- get
- set

미입력시 `get()`에 검증이 수행된다. 그렇기에 어떤 영역에 검증을 할 것인지 명시하자
![Pasted image 20251118170207](Pasted%20image%2020251118170207.png)

```kotlin
data class User(
	@field:NotNull(message = "생성한 유저 ID는 필수입니다")
	val id: Long
)
```

## 출처
[필드와 프로퍼티 차이점](https://bradkim94.medium.com/java-field-vs-kotlin-property-13b4ab2de830)
[코틀린 Validation 사용시 주의점](https://curiousjinan.tistory.com/entry/kotlin-spring-validation-setup-caution)
[프로퍼티와 필드의 차이점은 무엇이 있을까?](https://colabear754.tistory.com/164)
[백킹 필드란?](https://idea-beyond.tistory.com/206)
