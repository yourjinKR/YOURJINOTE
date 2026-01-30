# Dependency Injection

DI(의존관계 주입)는 객체가 의존하는 또 다른 객체를 외부에서 선언하고 이를 주입받아 사용하는 것이다.

## DI가 아닌 코드

```java
class BurgerChef {
    private HamBurgerRecipe hamBurgerRecipe;

    public BurgerChef() {
        hamBurgerRecipe = new HamBurgerRecipe();        
    }
}
```

## DI로 구현한 코드

```java
class BurgerChef {
    private BurgerRecipe burgerRecipe;

    public BurgerChef(BurgerRecipe burgerRecipe) {
        this.burgerRecipe = burgerRecipe;
    }
}
```

# 구현 방식

### 생성자

```java
class BurgerRestaurantOwner {
    private BurgerChef burgerChef = new BurgerChef(new HamburgerRecipe());

    public void changeMenu() {
        burgerChef = new BurgerChef(new CheeseBurgerRecipe());
    }
}
```

### 메서드

```java
class BurgerChef {
    private BurgerRecipe burgerRecipe = new HamburgerRecipe();

    public void setBurgerRecipe(BurgerRecipe burgerRecipe) {
        this.burgerRecipe = burgerRecipe;
    }
}

class BurgerRestaurantOwner {
    private BurgerChef burgerChef = new BurgerChef();

    public void changeMenu() {
        burgerChef.setBurgerRecipe(new CheeseBurgerRecipe());
    }
}
```

# 구현시 장점

- 의존성 감소 → 수정에 유연
- 재사용성 증가
- 테스트 용이
- 가독성

<br>

# 출처

[테코블](https://tecoble.techcourse.co.kr/post/2021-04-27-dependency-injection/)