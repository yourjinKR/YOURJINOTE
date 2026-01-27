
# SOLID

[객체 지향 프로그래밍](OOP.md) 및 설계에서 유지보수가 쉽고, 확장 가능하며, 읽기 쉬운 코드를 만들기 위한 5가지 기본 원칙(SRP, OCP, LSP, ISP, DIP)의 약자이다.

SOLID 원칙의 핵심은 **변경에 유연하고 이해하기 쉬운 소프트웨어 구조**를 만드는데 있다.

> 변경에 유연하다라는 말은 결합도는 낮고 응집도는 높은 구조를 의미한다.  

함수와 데이터 구조를 효과적으로 결합하고, 이 집합들을 서로 유기적으로 연결하는 방법을 제시한다.

- S : Single Responsibility Principle (단일 책임 원칙)
- O : Open-Closed Principle (개방-폐쇄 원칙)
- L : Liskov Substitution Principle (리스코프 치환 원칙)
- I : Interface Segregation Principle(인터페이스 분리 원칙)
- D : Dependency Inversion Principle(의존성 역전 원칙)

## Single Responsibility Principle (단일 책임 원칙)

단일 책임 원칙(SRP)이란 **하나의 클래스가 하나의 책임만을 가져야 한다**는 것을 의미한다.  
클래스가 변경되어야 하는 이유는 단 하나여야 하며, 이를 통해 코드의 **응집도를 높일** 수 있다.

### 준수 시 이점

- **테스트**: 책임이 하나인 클래스는 테스트 시나리오가 단순해져 테스트 케이스 작성이 훨씬 쉬워진다.
- **낮은 결합도**: 단일 클래스에 포함된 기능이 적어질수록 다른 클래스와의 불필요한 의존성이 줄어든다.
- **가독성과 구성**: 규모가 작고 명확한 이름을 가진 클래스는 거대한 클래스(God Object)보다 검색과 이해가 훨씬 빠르다.

### 예시 코드

```java
// bad: 사용자 정보 관리와 로그 출력을 모두 담당
class UserSettings {
    void changeUsername(String name) { /* 유저네임 변경 로직 */ }
    void logError(String error) { System.out.println(error); } // 다른 책임!
}

// good: 책임을 분리
class UserSettings {
    void changeUsername(String name) { /* 유저네임 변경 로직 */ }
}

class Logger {
    void logError(String error) { System.out.println(error); }
}
```

## Open-Closed Principle (개방-폐쇄 원칙)

개방-폐쇄 원칙(OCP)은 **소프트웨어 엔티티(클래스, 모듈 등)는 확장에는 열려 있어야 하고, 수정에는 닫혀 있어야 한다**는 원칙이다. 기존 코드를 변경하지 않고도 기능을 추가할 수 있어야 함을 뜻한다.

### 준수 시 이점

- **유연성**: 새로운 요구사항이 생길 때 기존 코드를 건드리지 않고 새로운 클래스를 추가하는 것만으로 대응 가능하다.
    
- **안정성**: 기존의 검증된 코드를 수정하지 않으므로, 수정으로 인해 발생할 수 있는 '사이드 이펙트(Side Effect)'를 방지한다.

### 예시 코드

```java
interface Shape {
    double calculateArea();
}

class Rectangle implements Shape {
    double width, height;
    public double calculateArea() { return width * height; }
}

class Circle implements Shape {
    double radius;
    public double calculateArea() { return Math.PI * radius * radius; }
}

// 새로운 도형이 추가되어도 AreaCalculator의 코드는 변하지 않음
class AreaCalculator {
    public double sum(Shape[] shapes) {
        double total = 0;
        for (Shape shape : shapes) total += shape.calculateArea();
        return total;
    }
}
```

## Liskov Substitution Principle (리스코프 치환 원칙)

리스코프 치환 원칙(LSP)은 **프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야 한다**는 원칙이다. 즉, 자식 클래스는 부모 클래스의 역할을 완전히 수행할 수 있어야 한다.

### 준수 시 이점

- **다형성 활용**: 부모 클래스 타입으로 작업하는 클라이언트 코드가 자식 클래스의 상세 구현을 몰라도 올바르게 동작함을 보장한다.
    
- **코드 예측 가능성**: 상속 구조에서 예상치 못한 동작을 방지하여 디버깅 시간을 줄여준다.

### 예시 코드

```java
class Bird {
    void fly() { /* 날기 구현 */ }
}

// 펭귄은 날 수 없으므로 Bird를 상속받으면 LSP 위반!
// 올바른 설계: 날 수 있는 새와 없는 새를 구분하거나 행동을 인터페이스로 분리
interface Flyable { 
	void fly(); 
}

class Sparrow extends Bird implements Flyable {
    public void fly() { /* 참새 비행 */ }
}

class Penguin extends Bird {
    // fly()를 구현하지 않음으로써 오해를 방지
}
```

## Interface Segregation Principle (인터페이스 분리 원칙)

인터페이스 분리 원칙(ISP)은 **특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 낫다**는 원칙이다. 사용하지 않는 메서드에 의존하도록 강제해서는 안 된다.

### 준수 시 이점

- **가벼운 구현**: 클래스가 자신에게 꼭 필요한 메서드만 구현하면 되므로 코드가 간결해진다.
    
- **재컴파일 방지**: 관련 없는 메서드의 변경으로 인해 해당 인터페이스를 사용하는 다른 클래스들을 다시 컴파일할 필요가 없어진다.

### 예시 코드

```java
// bad: 모든 기능을 한 인터페이스에 넣음
interface SmartPrinter {
    void print();
    void fax();
    void scan();
}

// good: 기능을 세분화
interface Printer { void print(); }
interface Fax { void fax(); }
interface Scanner { void scan(); }

class BasicPrinter implements Printer {
    public void print() { /* 인쇄만 수행 */ }
}
```

## Dependency Inversion Principle (의존역전 원칙)

의존역전 원칙(DIP)은 **추상화에 의존해야지, 구체화에 의존하면 안 된다**는 원칙이다. 고수준 모듈은 저수준 모듈의 구현에 휘둘리지 않아야 하며, 둘 다 추상화(인터페이스)에 의존해야 한다.

### 준수 시 이점

- **결합도 감소**: 특정 기술(DB 종류, 외부 라이브러리 등)에 종속되지 않는 핵심 비즈니스 로직을 작성할 수 있다.
    
- **테스트 용이성**: 실제 객체 대신 가짜 객체(Mock)를 주입하기 쉬워져 단위 테스트 효율이 극대화된다.

```java
interface Keyboard { void type(); }

class MechanicalKeyboard implements Keyboard {
    public void type() { System.out.println("찰칵찰칵"); }
}

class Computer {
    private final Keyboard keyboard;

    // 구체적인 클래스가 아닌 인터페이스(추상화)에 의존
    public Computer(Keyboard keyboard) {
        this.keyboard = keyboard;
    }
    
    void start() { keyboard.type(); }
}
```


<br>

# 출처

[카카오 뱅크 기술 블로그](https://tech.kakaobank.com/posts/2411-solid-truth-or-myths-for-developers/)  
[얄코 -  SOLID 원칙 - 객체지향 디자인 패턴의 기본기](https://www.youtube.com/watch?v=4O6k9GN8FPo)  


