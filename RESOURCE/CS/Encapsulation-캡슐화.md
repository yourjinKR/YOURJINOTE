# 캡슐화

**연관된 속성과 기능들을 하나의 캡슐**로 만들어 **데이터를 외부로부터 보호**나는 것을 말한다.
자바에서는 클래스에 정의된 속성과 기능들을 외부로부터 보호하고, 필요한 부분만 외부로 노출될 수 있도록 하여 각 **객체 고유의 독립성**과 **책임 영역**을 안전하게 지키고자 하는 목적이 있습니다.

## 캡슐화를 구현하기 위한 방법

보호 방법에는 접근 제어자를 활용하거나 getter/setter 메서드를 통해 구현합니다.

```java
public class Person {
    private String name; // 외부 접근 차단

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```