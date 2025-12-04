# 다형성
어떤 객체의 속성이나 기능이 그 맥락에 따라 다른 역할을 수행할수 있는 객체 지향의 특성을 의미한다.

## 부모 자식 클래스간 형식 변환
### 업 캐스팅
```java
Animal animal = new Animal();
Animal animal = new Monkey(); // 업 캐스팅
Animal animal = new Human(); // 업 캐스팅

Monkey monkey = new Animal(); // 불가
Monkey monkey = (Monkey) animal; // 가능
```
클래스의 모든 파생 형식 클래스는 해당 클래스로 캐스팅이 가능하다.
조상 자손 관계의 참조 변수는 서로 형변환이 가능하다.
> 실행하기 전까지 컴파일러가 인식하지 못하기에 사용에 주의할 필요가 있음.
### 다운 캐스팅
```java
Animal animal = new Monkey();  
System.out.println(animal.name());  
  
if (animal instanceof Monkey) {  
    ((Monkey) animal).tail();  
}
```
부모 클래스 형식 참조자가 가리키는 대상 인스턴스를 특정 파생 형식으로 캐스팅 하는 것이다.
`instanceof` 연산자를 통해 형변환 검사를 수행할 수 있다.
> 다운 캐스팅 또한 특수한 목적으로 사용 할 수는 있다.
> 그러나 다운 캐스팅 자체는 의존성이 역전되는 구조이기에 주의할 필요가 있다.

## 사용 이점
- 다형적 매개변수
- 하나의 배열로 여러 종류 객체를 다룰 수 있다
```java
class Product {
    int price = 100;
    public Product() {}
}

class Computer extends Product {}
class Mouse extends Product {}
class TV extends Product {}

// 모든 물건을 매개변수를 받을 수 있음
void buy(Product product) {
    money -=  product.price;
}

Product p[] = new Product[3];
p[0] = new Computer();
p[1] = new Mouse();
p[2] = new TV();
```

## 상속 관계에서 인스턴스 메모리 구조
![Pasted image 20251201183813](Pasted%20image%2020251201183813.png)
- 파생 클래스 인스턴스는 부모 클래스 인스턴스를 내부에 포함하고 있는 구조이다.
- 파생 클래스는 부모 클래스의 멤버를 가진 것이다.
	- 접근제어자에 따라 파생 관계에서 접근 권한을 관리할 수 있다. (private, protected)
