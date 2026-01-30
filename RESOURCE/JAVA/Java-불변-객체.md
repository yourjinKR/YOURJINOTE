# 불변 객체

- 모든 필드를 `final` 선언하여 상수화한 클래스 (무결성)
	- 필드 초깃값을 기술할 수 있는 부분 제외 (생성자 혹은 초깃값 정의)
- 필드 값을 변경시에는 새로운 사본 인스턴스를 수정된 초깃값으로 기술하여 반환 (원자성)
	- 해당 방식은 비효율적일 수 있으나 멀티스레딩 환경에서 장점이 있다.
- 대표 예시 : [String](Java-String.md), [Wrapper Class](Java-Wrapper-Class.md)

## 장점

### 스레드 안전성 (Thread-Safety)

가장 큰 장점입니다. 멀티스레드 환경에서 여러 스레드가 동시에 같은 객체에 접근하더라도, 상태가 절대 변하지 않기 때문에 **동기화(Synchronization) 처리가 필요 없습니다.** 데이터 오염 걱정 없이 안심하고 공유할 수 있죠.

### 보안 및 견고함 (Security & Robustness)

메소드 인자로 불변 객체를 전달했을 때, 외부(다른 코드)에서 내 객체의 값을 마음대로 바꿀 수 없음을 보장합니다. 이는 시스템의 **예측 가능성**을 높여줍니다.

### 캐싱 및 재사용성

값이 변하지 않으므로 마음놓고 캐싱할 수 있습니다. 예를 들어 Java의 `String Pool`처럼 동일한 값을 가진 객체를 재사용하여 **메모리 효율**을 높일 수 있습니다.

### 부수 효과(Side Effect) 방지

객체가 수정되지 않으니 함수 실행 시 외부 상태가 변하는 부수 효과가 줄어듭니다. 이는 코드의 유지보수성을 높이고 디버깅을 훨씬 수월하게 만듭니다.

## 구현 방식

```java
public class Cars {
    private final List<Car> cars;

    public Cars(List<Car> cars) {
    	this.cars = new ArrayList<>(cars);
    }

    public List<Car> getCars() {
        return Collections.unmodifiableList(cars);
    }
}
```

`final` 키워드만 붙였다고 해서 불변 객체가 될 수는 없다.  
`cars` 내부의 `car`를 꺼내와 변경하거나 값을 삽입/삭제한다면 사실상 `Cars`는 여전히 가변적이다.  
<br>

# 출처

[테코블](https://tecoble.techcourse.co.kr/post/2020-05-18-immutable-object/)  