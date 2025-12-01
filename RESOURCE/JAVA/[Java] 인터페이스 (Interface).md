# 인터페이스 (Interface)
```java
public interface repairable() {
	void repair();
}
```
## 인터페이스의 개념
- 추상 메서드의 집합, 필드를 가지지 않음
- 상수 필드는 가질 수 있다
- 모든 멤버가 `public`
## 인터페이스의 구현
```java
public class Robot implements Repairable {  
    @Override  
    public void repair() {  
        System.out.println("로봇을 수리함");  
    }  
}

public class RepairShop {  
    public void repair(Repairable repairable) {  
        repairable.repair();  
    }  
}
```
- `implements` 키워드를 통해 구현한다.
- **다중 상속이 가능**하다.
- 생성자를 가질 수 없다.
- 인터페이스에 명시된 추상 메서드를 전부 구현해야 된다.
## 인터페이스의 목적
- 객체 간의 연결을 돕는 중간 역할을 수행한다.
- 선언과 구현을 분리할 수 있다.
- 서로 관계가 없는 클래스들 간에 관계를 맺어줄 수 있다.

## default 메서드와 static 메서드
- Java 8부터 추가
- 인터페이스에 디폴트 메서드, static 메서드 추가 가능
- 디폴트 메서드는 인스턴스 메서드 (인터페이스 원칙을 위반함)
```java
interface MyInterface {
	void method();
	// 아맞다이거추가해야되는데충돌문제생길거같으니깐강제성이없음
	default void method() {}; // deafault 접근 제어자 추가
}
```
### 예시
```java
public interface Repairable {  
    void repair();  
  
    default void repairByMoney(int money) {  
        if (money > 1000) {  
            repair();  
        }  
    }  
}
```