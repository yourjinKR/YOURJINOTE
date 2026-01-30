# 추상 클래스와 인터페이스의 공통점

- 인스턴스(객체)를 생성할 수 없다.
- 선언만 있고 구현 내용이 없다.
- 자식클래스에게 메서드의 구체적인 동작을 구현하도록 책임을 위임한다.

# 추상 클래스와 인터페이스의 차이점

[추상 클래스](Java-추상-클래스-Abstract-Class.md)는 추상메서드를 자식 클래스가 구체화하여 그 기능을 확장하는데 목적이있다. (상속을 받아 기능을 확장시키는것) 하지만, [인터페이스](Java-인터페이스-Interface.md)는 서로 관련이 없는 클래스에서 공통적으로 사용하는 방식이 필요하지만 각각의 기능을 구현할 필요가 있는경우에 사용한다.

- **추상 클래스**는 클래스이지만 **인터페이스**는 클래스가 아니다.
- **추상 클래스**는 단일 상속이지만 **인터페이스는** 다중상속이 가능하다.

![추상클래스와 인터페이스 그림](../../Excalidraw/추상클래스와%20인터페이스%20그림.md)

# 출처

[geeksforgeeks - difference-between-abstract-class-and-interface-in-java](https://www.geeksforgeeks.org/java/difference-between-abstract-class-and-interface-in-java/)  
[티스토리 - 추상 클래스와 인터페이스의 차이점](https://jtm0609.tistory.com/175)  