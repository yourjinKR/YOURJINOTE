# Comparable vs Comparator

- Comparable : 클래스 내 객체들의 자연스러운 순서를 정의하는데 사용하는 인터페이스
- Comparator : 외부에서 사용자 지정 정렬 로직을 정의하는데 사용

## Comparator

- 클래스 내 객체들의 자연스러운 순서를 정의하는데 사용하는 인터페이스
- `compareTo()`를 재정의

> 객체 내부에서 선언하기에 `compareTo()` 메서드에서  `this`와 매개변수를 비교함

### 예시

```java
class Person implements Comparable<Person> {  
    int age;  
  
    public Person(int age) {  
        this.age = age;  
    }  
  
    @Override  
    public int compareTo(Person o) {  
        // return this.age > o.age ? 1 : this.age == o.age ? 0 : -1;  
        return Integer.compare(this.age, o.age);  
    }  
  
    @Override  
    public String toString() {  
        return "Person{" +  
                "age=" + age +  
                '}';  
    }  
}
```

age와 같은 단순한 숫자 형식은 아래와 같이도 비교할 수도 있다.

```java
return this.age - o.age;
```

```java
int this.data = Integer.MAX_VALUE; //  2_147_483_647
int o.data = -1;

return this.data - o.data;
```

그러나 위와 같이 숫자가 크거나 예측할 수 없는 범위라면 오버플로우의 위험성이 존재한다.


## Comparator

- 외부에서 사용자 지정 정렬 로직을 정의하는데 사용
- `compare()` 메서드를 재정의

> 외부에서 선언하기에 기본적으로 기본 메서드에 두개의 매개변수가 있음

```java
public class Ranking implements Comparator<Player> {  
    @Override  
    public int compare(Player o1, Player o2) {  
        return Integer.compare(o1.level, o2.level);  
    }  
}
```

```java
List<Player> players = new ArrayList<>();  
players.add(new Player("마법사", 12, 30, 5));  
players.add(new Player("도적" ,11, 25, 20));  
players.add(new Player("궁수" ,13, 12, 30));  
players.add(new Player("싸움꾼", 15, 6, 50));  
players.add(new Player("기사", 11, 3, 100));  
  
Ranking ranking = new Ranking();  
players.sort(ranking);  
System.out.println("레벨 순 : " + players);  
// 레벨 순 : [도적, 기사, 마법사, 궁수, 싸움꾼]
```

# 출처

[geeksforgeeks](https://www.geeksforgeeks.org/java/comparable-vs-comparator-in-java/)  