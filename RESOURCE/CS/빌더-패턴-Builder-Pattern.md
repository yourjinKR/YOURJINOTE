# 개념
**빌더**는 복잡한 객체들을 단계별로 생성할 수 있도록 하는 생성 디자인 패턴입니다. 이 패턴을 사용하면 같은 제작 코드를 사용하여 객체의 다양한 유형들과 표현을 제작할 수 있습니다.

# 장점
- 필요한 데이터만 설정할 수 있음
- 유연성을 확보할 수 있음
- 가독성을 높일 수 있음
- 변경 가능성을 최소화할 수 있음

# 자바 Builder
## 직접 구현
```java
public class User {

  private Long id;
  private String email;
  private String password;
  private String name;

  public User(Long id, String email, String password) {
    this.id = id;
    this.email = email;
    this.password = password;
  }

  ...getter 함수

  // ✨ 빌더 패턴을 위한 빌더 클래스!
  static public class Builder {
    private Long id;
    private String email;
    private String password;

    public Builder() {
    }

    public Builder(User user) {
      this.id = user.id;
      this.email = user.email;
      this.password = user.password;
    }

    public Builder id(Long id) {
      this.id = id;
      return this;
    }

    public Builder email(String email) {
      this.email = email;
      return this;
    }

    public Builder password(String password) {
      this.password = password;
      return this;
    }

    public User build() {
      return new User(id, email, password);
    }
  }
}
```
## Lombok
```java
@Getter @Builder // ✨ 클래스 전체 필드를 빌더로 사용 가능!
public class UserLombok {

  private Long id;
  private String email;
  private String password;
  private String name;
}

// 사용예제
public User join(String email, String password, String name) {
  UserLombok build = UserLombok.builder()
            .email(email)
            .password(password)
            .name(name)
            .build();
  ...
}
```

### 세부기능
```java
@Builder(  
        // true시 이미 생성된 객체의 값을 기반으로 새로운 빌더 인스턴스를 생성  
        toBuilder = true,  
        // 기본값 "builder 예 : User.builder()"  
        builderMethodName = "create",  
        // 기본값 "build" 예 : User.builder().build();  
        buildMethodName = "done",  
        // 기본값: "" (없음). 필드명 그대로 사용 (예: .name("지민"))  
        setterPrefix = "with",  
        // 기본값 : ClassNameBuilder  
        builderClassName = "UserBuilder123",  
        // access  
        access = AccessLevel.PUBLIC  
)
```

![Pasted image 20251130020501](../../GALLERY/Pasted%20image%2020251130020501.png)

### @Builder.Default
```java
@Builder.Default  
private boolean hidden = false;
```
빌더에 값을 할당하지 않아도 기본값을 설정해준다.
필드에 명시하여 사용

### @Builder.ObtainVia
```java
@Getter
@Builder(toBuilder = true)
@NoArgsConstructor
@AllArgsConstructor
public class Person {

    private String firstName;
    private String lastName;
    private int age;

    @Builder.ObtainVia(method = "calculateFullName")
    private String fullName;

    public String calculateFullName() {[
    
        if (this.fullName != null) {
            return this.fullName;
        }
        
        return String.join(firstName, " ", lastName);]()
    }
}
```
해당 어노테이션을 필드에 명시하면 필드/매개변수를 가져오는 대체 수단을 나타낼 수 있다
메서드를 명시하면 해당 메서드를 통해 값이 초기화된다.

```java
@Test
void BuilderPatternTest1() {
    
	Person person = Person.builder()
			.firstName("John")
			.lastName("Doe")
			.age(30)
			.build()
			.toBuild()
			.build();

	assertThat(person.getFullName).isEqualTo("John Doe"); 
	// result : success (true)
}
```

# 출처
[망나니 개발자님 블로그 - 자바 빌더 패턴](https://mangkyu.tistory.com/163)

[빌더 패턴의 개념](https://refactoring.guru/ko/design-patterns/builder)