# Generic

제네릭은 **매개변수화 된 타입**을 의미한다. 이를 통해 클래스, 인터페이스 또는 메서드를사용하여 다양한 데이터 타입을 처리하는 코드를 작성할 수 있다. 이를통해 **코드의 재사용성**과 **타입 안정성**을 **확보**한다.

- 매개변수화된 타입
- 컴파일 시에 타입을 체크하는 기능
- 타입 안정성을 높이고 형변환이 필요 없음

제네릭을 왜 사용하는가에 대한 글은 [여기](제네릭을%20사용하는%20이유에%20대해%20설명해주세요.md)를 참고

## 타입 매개변수 네미밍 컨벤션

- T - 유형
- E - 요소 (컬렉션 프레임워크에서 주로 사용됨)
- K - 키
- N - 숫자
- V - 값
- SUV, S, V 등 - 2세대, 3세대, 4세대

## 제네릭 클래스

클래스를 정의할 때, Object 타입 대신 `T`와 같은 타입변수를 사용

```java
public class Box<T> {  
    public T item;  
  
    public T getItem() {  
        return item;  
    }  
  
    public void setItem(T item) {  
        this.item = item;  
    }  
}
```

- type parameter (타입 매개변수) : `class Box<T>`의 `T`

```java
Box<Apple> appleBox = new Box<Apple>();
```

- type argument (타입 인자) : `Box<Apple> appleBox` 의 `Apple` 

### 확장 관계

```java
Box<Apple> appleBox = new Box<>();  
appleBox.setItem(new Apple());  
// appleBox.setItem(new Fruit()); 불가능!!!  
  
Box<Fruit> fruitBox = new Box<>();  
fruitBox.setItem(new Apple());
```

```java
static class FruitBox<T extends Fruit> {  
    public ArrayList<T> items = new ArrayList<>();  
}
```

제네릭 타입에 `extends` 키워드로 특정 타입의 자식 클래스들로만 제한할 수 있다.

> 인터페이스 또한 `implements` 키워드가 아닌 `extends` 키워드를 통해 제네릭 타입을 제한 할  수 있다. 


![Pasted image 20251224142234](../../GALLERY/Pasted%20image%2020251224142234.png)

> 그림은 예시와 다르지만 참고하자.

```java
Apple apple = new Apple();  
Fruit fruit = apple;  
  
Box<Apple> appleBox = new Box<>();  
Box<Fruit> fruitBox = appleBox1; // 불가능
```

`Apple`은 `Fruit`의 하위 유형이기에 `fruit`에 `apple`를 할당할 수 있다.
그러나 `fruitBox`에 `appleBox`는 할당 할 수 없다.  
그 이유는 애초에 `FruitBox<Apple>`은 `FruitBox<Box>`의 하위 유형이 아니기 때문이다.  

> 제네릭은 서로 형변환이 불가능하다.

이와 같은 경우에는 와일드 카드를 활용하여 해결 가능하다.

## 와일드 카드

```java
Box<? extends Fruit> fruitBox2 = appleBox1;
```

와일드 카드 '`?`' 를 사용하면 제네릭  간 형변환이 가능하다.

- `<?>` : 제한 없음 (like `Object`)
- `<? extends 부모타입>` : 상한
- `<? super 자식타입>` : 하한

와일드 카드는 클래스나 메서드 선언시에는 사용이 불가능하다.

> 와일드카드는 설계가 아닌 사용을 위한 개념이다.

### Map.java의 comparingByKey()
```java
public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K, V>> comparingByKey() {  
    return (Comparator<Map.Entry<K, V>> & Serializable)  
        (c1, c2) -> c1.getKey().compareTo(c2.getKey());  
}
```

- 이 메서드는 타입 매개변수 K, V를 가지는 제네릭 메서드이다.  
- K는 Comparable을 구현한 타입만 허용된다. 즉, K 타입 객체는 compareTo() 메서드를 호출할 수 있어야 한다.  
- Comparable 내부의 K는 K 또는 K의 부모 타입과 비교 가능하다.
- 반환 타입은 Map.Entry<K,V>를 비교하는 Comparator를 반환한다.

## 제네릭 메서드

```java
static public <V> V process(V data) {  
    return data;  
}
```

아래 예시를 통해 해당 메서드가 제네릭 메서드인지에 대해 잘 구분하자.

```java
public class MethodEx<T> {  
  
    // 클래스 제네릭 타입을 사용하는 메서드  
    T test(T data) {  
        return data;  
    }  
  
    // 제네릭 메서드  
    static public <V> V process(V data) {  
        return data;  
    }  
  
    public static void main(String[] args) {  
        MethodEx<String> example = new MethodEx<String>();  
          
        System.out.println(example.test("데이터"));  
        System.out.println(MethodEx.<String>process("데이터"));  
    }  
}
```

결정적인 차이는 타입 결정 시점이다.

### 주의 사항

```java
public class Example<T> {
    <T> T method(T t) { ... }
}
```

클래스의 `T`와 메서드의 `T`는 별개이다, 그렇기에 네이밍을 구분하도록 하자.

## 타입 추론

제네릭의 [타입 추론](Java-타입-추론.md)은 컴파일러가 메서드 호출 시점의 인자, 컨텍스트를 기반으로 제네릭 타입 매개변수 `<T>`를 자동으로 결정하는 기능이다.

### 클래스

```java
Box<Fruit> fruitBox = new Box<Fruit>();  
  
// 생략 가능  
Box<Fruit> fruitBox2 = new Box<>();
```

### 메서드

```java
class Util {  
    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {  
        return p1.getKey().equals(p2.getKey()) &&  
                p1.getValue().equals(p2.getValue());  
    }  
}  
  
class Pair<K, V> {  
    private K key;  
    private V value;   
    // 생성자, getter/setter 생략
}
```

```java
Pair<Integer, String> p1 = new Pair<>(1, "apple");  
Pair<Integer, String> p2 = new Pair<>(2, "pear");  
  
// 아래 <Integer, String>는 생략해도 된다 (타입 추론)  
boolean same = Util.<Integer, String>compare(p1, p2);
```

## 멀티 타입 파라미터 제네릭

```java
class Board<K, V> {  
    public Map<K, V> data;  
  
    public Board(Map<K, V> data) {  
        this.data = data;  
    }  
}
```

```java
Map<Integer, String> data = new HashMap<>();  
Board<Integer, String> board = new Board<>(data);  
  
board.data.put(1, "유어진");  
board.data.put(2, "유재석");  
board.data.put(3, "유지민");  
```

## 주의사항

### 참조 형식에서만 작동

```java
List<int> intList1 = new ArrayList<>(); // 불가능!  
List<Integer> intList2 = new ArrayList<>();
```

### 타입 소거

타입 소거란 제네릭 타입 정보가 컴파일 시에만 존재하고, 컴파일이 끝나면 바이트 코드에서는 제거되는 것을 말한다.  

이로 인해 아래와 같은 제약 사항이 있다.

- `instanceof`에 제네릭 타입 사용 불가
- 제네릭 타입으로 객체 생성 불가
- 제네릭 배열 생성 불가
- 제네릭 타입 정보로 메서드 오버로딩 불가
- `static` 필드에 타입 매개변수 사용 불가

```java
// 전
List<String> list = new ArrayList<>();
// 후
List list = new ArrayList();
```

```java
// 전
class Box<T extends Number> {
    T value;
}
// 후
class Box {
    Number value;
}
```

```java
// 전
T get() {
    return value;
}
// 후
Object get() {
    return (Object) value;
}
```

#### 브릿지 메서드

브릿지 메서드란 타입 소거로 인한 다형성이 깨지는 것을 방지하여 컴파일러가 생성하는 메서드이다.

```java
class Parent<T> {
    T get() { return null; }
}

class Child extends Parent<String> {
    @Override
    String get() { return "child"; }
}
```

```java
String get() { return "child"; }

Object get() { return get(); } // 브리지 메서드
```

```java
Parent p = new Child();
Object o = p.get();
```

#### 왜 타입 소거가 되는가

자바는 제네릭 도입 시 다음 요구사항을 준수해야 했다.

- 기존 코드와의 바이너리 호환성 유지
- 제네릭이 없는 JVM에서도 동작
- 런타임 비용 증가 방지

<br>

# 출처

[Oracle - Generic Inheritance](https://docs.oracle.com/javase/tutorial/java/generics/inheritance.html)  
[지마켓 기술 블로그 - 제네릭 심화](https://dev.gmarket.com/28)  
[남궁성 - 자바의 정석](https://www.youtube.com/watch?v=GsCiLF-o2aI&list=PLW2UjW795-f4dHD3VADsvH7ZkPuFAF7Vo&index=70)  
[유튜브 - 테코톡](https://www.youtube.com/watch?v=Cr6mlXWZ35E)  

<br>

# 다시 점검할 내용

[기술면접-Java-Generic](기술면접-Java-Generic.md)  