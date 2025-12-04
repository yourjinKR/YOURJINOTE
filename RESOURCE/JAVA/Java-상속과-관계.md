# 상속
```java
class Child extends Parent { }
```
- 객체 단위 코드를 재사용(확장)하는 문법
- is-a, has-a 관계로 설명 가능
- 부모/자식 클래스, 기본/파생 클래스로 불린다.
- 다중상속은 허용하지 않는다
- Java에서 모든 클래스는 `Object`의 파생 형식

## 기본 이론
- 파생 클래스 생성 시 기본 클래스도 함께 생성 (생성자 호출)
- 파생 클래스는 기본 클래스 멤버에 접근 가능
	- 접근제어 지시자에 따라 제어
- 파생 클래스에서 기본 클래스 메서드 재정의(Override) 가능
- 추상 자료형 개념 적용 가능
- 상속 관계가 적용될 경우 흐름이 **2차원적 구조**를 갖게 되어 감춰지는 경향이 있음
	- 다른 시점의 작성자와 소통하는 개념

## 상속과 생성자 호출스택
- 파생 클래스 생성자는 가장 먼저 **호출**되지만 가장 나중에 **실행**됨
- 재귀호출과 비슷하게 생성자 함수에 대한 호출 스택이 쌓여 올라가는 것이 특징임
> 파생 클래스에서 기본 클래스 필드를 정의하는 것은 적절하지 않은 경우가 대부분

<br/>

![Pasted image 20251129161824](../../GALLERY/Pasted%20image%2020251129161824.png)

## 상속을 배우는 순간부터 알아야 할 시간차 
- 부모 클래스와 파생 클래스가 정의되는 시점
	- 부모 클래스 기준 ‘현재’는 파생 클래스가 ‘미래’에 정의되는 것임을 확정 
	- 6개월 후 미래의 당신은 오늘 이 클래스에 대해 얼마나 기억 할 수 있을 것인지 고려
- 이해도 부족에 따른 문제
	- 증조 할아버지에 대해 얼마나 알고 있는가?
	- 상속에 상속이 이어지는 것은 결코 바람직하지 않음
- 미래의 파생 클래스를 고려해 현재 클래스 코드 작성

## super()
부모의 멤버를 접근할때 사용
```java
class Parent {
	int age;
}

class Child {
	int getMyParentAge(int age) {
		System.out.println(age);
		System.out.println(this.age);
		System.out.println(super.age);
	}
}
```

## 메서드 재정의 (Override)
- 파생 클래스에서 기본 클래스 메서드 재정의 가능
	- `@Override 어노테이션`
- 기본 클래스 메서드를 재정의 하는 것은 기존 메서드를 대체하거나 코드를 추가할 목적으로 볼 수 있다
- **메서드 재정의 시 실제 인스턴스 형식이 우선**
- `final` 선언으로 파생 클래스에서 메서드가 재정의되지 못하도록 차단할 수 있음

```java
public class Animal {  
    public String name() {  
        return "동물";  
    }  
}

public class Lion extends Animal {  
    @Override  
    public String name() {  
        return "사자";  
    }  
}

Animal animal = new Lion();  
System.out.println(animal.name());
// "사자" 출력
```

## 불필요한 호출로 미래와 대화하기
- 부모 클래스에서 정한 함수 호출 관계로 파생 클래스에서 재정의된 메서드가 호출 될 수 있음
- 구조가 정한 흐름을 확장해 (그 구조에) 내 코드를 추가하는 방식으로 올라 탈 수 있음
	- **Called by framework - onXxx() 메서드**
### Called By Framework
```java
class Version {  
    protected String nickname;  
  
    public void setNickname(String nickname) {  
        this.nickname = onSetNickName(nickname) ? nickname : "부적절한 닉네임";  
    }  
  
    protected boolean onSetNickName(String nickname) {  
        return true;  
    }  
}  
  
class Version2 extends Version {  
    @Override  
    protected boolean onSetNickName(String nickname) {  
        return nickname.length() <= 10;  
    }  
}

public class CallByFramework {  
    public static void main(String[] args) {  
        Version version = new Version2();  
        version.setNickname("제한글자가 열글자 이상이면 부적절한 닉네임입니다");  
        System.out.println(version.nickname);  
    }  
}  
```