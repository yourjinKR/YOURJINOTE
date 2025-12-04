# Interface에 대한 이슈 정리
### 언제
2025-11-05, 우테코 내부 온라인 스터디 정기회의 중
### 왜
온라인 방식의 발표가 미숙하여 제대로 의미전달이 안된건지 스터디원께서 Interface 사용에 대해 의문을 제기하셨습니다. 아주 잘 사용했다고 뿌듯하고 있었는데 정반대의 의견이 나와 당황했지만 제가 사용했던 이유를 제대로 설명하지 못했습니다.
## 저의 의도
이번 로또 뽑기 과제 설계 과정 중 도메인을 정의할때 내린 판단의 근거는 아래와 같습니다.
### 도메인 정의
로또는 다양한 `규칙`을 갖고 있다. 그 중에서도 해당 문제에서 설명하는 로또를 파악해보자.  
사용자는 `특정 번호`를 `추첨`을 통해 얻게 된다.  
해당 번호가 `당첨 번호`와 `보너스 번호`같은 기준을 얼마나 충족시켰는가로 여러 `당첨`결과가 있다.
- 특정번호 : `Lotto`
- 당첨 번호 & 보너스 번호 : `WinningLotto`
- 추첨 : `PickRule`
- 당첨 : `Winning`
- 규칙 : `GameRule`
	- `PickRule`과 `List<Winning>`를 관리
### PickRule
```java
public interface PickRule<T> {  
    public T pick();  
}
```
- 추첨 방식을 interface로 추상화 했다고 생각하시면 됩니다.
- 추첨 방식이 수행해야 되는 행위는 당연히 추첨이기에 이를 나타내고자 `pick()` 메서드를 추가

> 추첨방식은 추첨방식인데,,, 어떤 방식으로 추첨할 것이냐?
### LottoPickRule
```java
public class LottoPickRule implements PickRule<List<Integer>> {
	@Override  
	public List<Integer> pick() {  
	    return Randoms.pickUniqueNumbersInRange(startInclusive, endInclusive, count);  
	}
}
```
- 바로 숫자 리스트를 뽑아야겠죠

### PlayRule
```java
public class PlayRule {  
    public static final int PRICE = 1_000;  
    private final PickRule<List<Integer>> pickRule;  
    private final Map<Integer, List<Winning>> winningRuleSet = new HashMap<>();
    
}
```
- 해당 객체는 `PlayRule`내부 에서 관리합니다.
- 숫자 리스트를 뽑되 세부적으로 어떻게 뽑는가에 대해 


> playRule은 `규칙`이라고 언급했지만 `진행방식`, `운영방식`과 같은 넓은 개념을 객체로 만들었습니다.

## 팀원의 의견
> `PickRule<List<Integer>>`
> 해당 방식은 interface를 사용하는 방식과 어긋난듯 하다, 유연하지 않다.

라는 리뷰를 받았습니다.

인터페이스에 대해 한번 더 알아봅시다.

[출처](https://f-lab.kr/insight/understanding-java-interfaces-and-polymorphism)
### 자바 인터페이스 소개
자바에서 인터페이스는 객체의 사용 방법을 정의하는 데 사용되는 추상 타입입니다. 인터페이스는 메서드의 시그니처만을 포함하며, 이를 구현하는 클래스는 인터페이스에 선언된 모든 메서드를 구현해야 합니다. 인터페이스는 자바의 다형성을 실현하는 중요한 메커니즘 중 하나로, 다양한 클래스가 같은 인터페이스를 구현함으로써 동일한 방식으로 사용될 수 있습니다.

인터페이스는 다형성을 실현하고 코드의 유연성과 재사용성을 향상시키는 도구이다.
다양한 클래스를 동일한 방식으로 처리할 수 있기 때문이다.

### 이렇게 구현?
```java
public final class LottoPickRule implements PickRule<Lotto> {
    private final NumberGenerator generator; // Randoms 감싼 포트(테스트 더블 주입 용이)

    public LottoPickRule(NumberGenerator generator) {
        this.generator = generator;
    }

    @Override
    public Lotto pick() {
        List<Integer> nums = generator.pickUnique(Lotto.START_INCLUSIVE, Lotto.END_INCLUSIVE, Lotto.COUNT);
        return new Lotto(nums); // 생성 시점에 불변식 검증
    }
}
```



