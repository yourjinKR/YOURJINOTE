# 중첩 클래스 (Inner Class)

- 클래스 내부에 선언된 클래스
- 아래와 같이 구분된다
	- 정적 중첩 클래스
	- 내부 클래스 (비정적 중첩 클래스)

## 사용 목적

- 특정 위치에서만 사용되는 클래스들을 **논리적으로 그룹화**
- 더욱 견고한 **캡슐화**
- **읽기 쉬운 코드** 작성을 통한 **유지보수성** 증가[

## Static Nasted Class

- 외부 클래스에 속했을 뿐 사실 상 독립적인 다른 클래스
- 외부 클래스의 `private` 졍적 멤버는 접근 가능
- 외부 클래스의 `private` 인스턴스 멤버는 접근 불가

```java
public class OuterClass {
    private static String outerStaticData = "바깥 정적 데이터";
    private String outerInstanceData = "바깥 인스턴스 데이터";

    // 1. 정적 중첩 클래스
    public static class StaticNestedClass {
        public void display() {
            // 바깥 클래스의 정적 멤버에는 직접 접근 가능
            System.out.println("정적 중첩 클래스: " + outerStaticData);
            // 바깥 클래스의 인스턴스 멤버에는 직접 접근 불가능 (컴파일 오류 발생)
            // System.out.println("접근 시도: " + outerInstanceData); 
        }
    }
}

// 사용 예시
public class Main {
    public static void main(String[] args) {
        // 바깥 클래스 객체 생성 없이 바로 접근 및 생성 가능
        OuterClass.StaticNestedClass nested = new OuterClass.StaticNestedClass();
        nested.display(); // 출력: 정적 중첩 클래스: 바깥 정적 데이터
    }
}
```


## Non-static (Inner Class)

- 외부 클래스의 일부 요소로 속하는 클래스
	- 외부 클래스 내부에 정의
- 내부 클래스는 외부 클래스의 `private` 멤버에 대해 접근이 허용됨  

- 아래와 같이 3가지로 구분
	- Inner Class
	- Local Class
	- anonymous Class (익명 클래스)

### Inner Class

- 외부(Outer) 클래스의 일부요소로 속하는 클래스
	- 외부 클래스 내부에 정의
- 내부 클래스는 외부 클래스의 private 멤버에 대해 접근이 허용됨

```java
class UserData {  

    UserData(String name, String addr, String phone) {  
        this.name = name;  
        info = new Address(addr, phone);  
    }  
  
    private String name;  
    private Address info;  
  
    public Address getInfo() {  
        return info;  
    }  
  
    class Address {  
        public Address(String addr, String phone) {  
            this.addr = addr;  
            this.phone = phone;  
        }  
  
        private String addr;  
        private String phone;  
  
        public String getUserInfo() {  
            return UserData.this.name + ", " + addr + ", " + phone;  
        }  
  
    }  
    
}
```

```java
public class Main {  

    public static void main(String[] args) {  
        UserData user = new UserData(  
                "Hosung"  
                ,  
                "Seoul"  
                , "010-1111-111");  
        System.out.println(user.getInfo().getUserInfo());  
        UserData.Address addr = user.new Address(  
                "Hanam"  
                , "010-2222-2222");  
        System.out.println(addr.getUserInfo());  
    }  
    
}
```

### Local Class

- 메서드 바디 스코프 내부에 정의하는 클래스
	- 익명 객체를 이해하기 위한 필수 이론
- 메서드 바디 지역변수에 접근 가능
	- `final` 혹은 유사 `final` 변수
	- 스택은 객체 인스턴스보다 수명이 길 수 있다

```java
public class Main {  

    static void testFunc(Object obj) {  
        System.out.println(obj.toString());  
    }  
    
    public static void main(String[] args) {  
        int local = 20;  
        
        class LocalClass {  
            LocalClass(){ data = 10; }  
            int data;  
            void printData() {  
                System.out.println(this.data);  
                System.out.println(local);  
            }  
        }  
        
        LocalClass localClass = new LocalClass();  
        testFunc(localClass);  
        localClass.printData();  
    }  
    
}
```

### Inner Interface

- 클래스 내부에서 interface를 선언하는 경우
- 주로 정적 멤버 인터페이스로 선언하고 활용
	- 이벤트 드라이븐 구조를 갖는 **GUI 개발에 자주 등장**
	- 마우스 클릭 이벤트 발생 시 이를 처리해야 하기 위한 이벤트 감지(Listener) 코드와 대응(Handler) 코드를 일정형식으로 제한하기 용이 (GUI framework의 보편적 형태)

**좌표**

```java
class MyPoint {  
    MyPoint(int x, int y) {  
        this.x = x;  
        this.y = y;  
    }  
    int x;  
    int y;  
}
```

**마우스 이벤트 처리기를 포함한 윈도우 객체**

```java
class MyWindow {  
    static interface OnClickListener {  
        public void onClick(MyPoint point);  
    }  
    OnClickListener listener;  
    MyWindow(OnClickListener listener) {  
        this.listener = listener;  
    }  
    void click(MyPoint point) {  
        listener.onClick(point);  
    }  
}
```

**이벤트 리스너만 재정의 후 코드 작성**

```java
class ButtonListener implements MyWindow.OnClickListener {  
    @Override  
    public void onClick(MyPoint point) {  
        System.out.print("ButtonListener.onClick(): ");  
        System.out.println(point.x + ", " + point.y);  
    }  
}  
public class Main {  
    public static void main(String[] args) {  
        MyWindow win = new MyWindow(new ButtonListener());  
        win.click(new MyPoint(10, 10));  
        win.click(new MyPoint(200, 150));  
    }  
}
```

## 중첩 클래스의 사용 예시 : 빌더 패턴

```java
// Spring/JPA 엔티티 또는 DTO 예시
public class Member {
    private final Long id;
    private final String name;
    private final String email;

    // 생성자를 private으로 만들어 Builder를 강제
    private Member(Builder builder) {
        this.id = builder.id;
        this.name = builder.name;
        this.email = builder.email;
    }

    // 객체 생성을 위한 정적 중첩 클래스 (Builder 패턴)
    public static class Builder {
        private Long id;
        private String name;
        private String email;

        public Builder id(Long id) { this.id = id; return this; }
        public Builder name(String name) { this.name = name; return this; }
        public Builder email(String email) { this.email = email; return this; }

        public Member build() {
            return new Member(this);
        }
    }
    
    // ... getter 메서드 생략 ...
}

// 사용 예시 (서비스 계층 등)
Member newMember = new Member.Builder()
							.id(1L)
							.name("유어진")
							.email("you@gmail.com")
							.build();
```


# 익명 클래스

익명 클래스는 이름이 없는 내부 클래스이다.  
단 하나의 객체만 필요할 때, 클래스 선언과 객체 생성을 동시에 할 수 있다.

```java
interface Greeting {
    void sayHello();
}

public class Main {
    public static void main(String[] args) {
        final String name = "World"; // 사실상 final

        // Greeting 인터페이스를 구현하는 익명 클래스
        Greeting englishGreeting = new Greeting() {
            @Override
            public void sayHello() {
                System.out.println("Hello, " + name + "!");
            }
        };

        englishGreeting.sayHello(); // 출력: Hello, World!
    }
}
```

- 특정 클래스나 인터페이스의 파생 형식으로 존재
- 클래스로 정의해야 할 필요가 있는 대상이지만 특정구간에서만 필요할 뿐 재사용 가능성이 없을 때 유용
- 생성자 없음
- 하나의 구문에 클래스가 포함되는 구조
	- GUI framework에서 이벤트 수신 및 처리 코드를 효율적으로 간소화

> Java 8 이후 람다의 등장으로 사용 빈도 수가 크게 줄었다.

### 익명객체와 람다식 예시

```java
// Spring Boot 애플리케이션의 초기화 시점 등에서 Runnable을 익명 클래스로 사용하던 예시
public class LegacyRunner {
    public void runTask() {
        // Runnable 인터페이스를 익명 클래스로 구현
        Thread worker = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("익명 클래스를 사용하는 비동기 작업 실행...");
                // Spring Bean 호출 등 작업 수행
            }
        });
        worker.start();
    }
}
```

```java
// 람다식을 활용한 간단한 구현
Thread worker = new Thread(() -> {
    System.out.println("람다 표현식을 사용하는 비동기 작업 실행...");
});
```


# 출처
[인프런 - 널널한 개발자](https://www.inflearn.com/course/%EB%8F%85%ED%95%98%EA%B2%8C-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-java-part2/dashboard)  
[티스토리 - 자바 내부 클래스 개념 정리 및 사용법](https://ittrue.tistory.com/123)  