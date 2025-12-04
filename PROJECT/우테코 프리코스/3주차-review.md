
# PR 코멘트
- 작업기간 : 10/31 ~ 11/02
- 도메인을 먼저 설계 후 진행
### 도메인 정의
로또는 다양한 규칙을 갖고 있다. 그 중에서도 해당 문제에서 설명하는 로또를 파악해보자.
**특정 번호**들을 사용자가 얻는 방식인 **추첨**과 해당 번호가 **당첨 번호**와 **보너스 번호**같은 기준을 얼마나 충족시켰는가로 여러 **당첨**결과가 있다.

- 특정번호 : `Lotto`
- 당첨번호, 보너스 번호 : `WinningLotto`
- 추첨 : `PickRule`
- 당첨 : `Winning`
- 규칙 : `PickRule`과 `List<Winning>`를 관리
### 고민점
이번 과제를 진행하며 어려움을 느꼈던 부분을 아래 서술하겠습니다.
#### 발행된 로또 번호와 당첨 번호 비교 로직
아래와 같이 구현했으나 코드만 봤을때는 의미전달이 부족하다고 느꼈습니다.

```
5개 일치 (1,500,000원) - 0개 
5개 일치, 보너스 볼 일치 (30,000,000원) - 0개
```
숫자 5개를 맞출 경우에는 보너스 일치 여부에 따라 상금이 달라진다.

```
4개 일치 (50,000원) - 0개
```
그 외 경우에는 보너스 일치 여부와 상관 없이 상금이 동일함.

각각의 로또 번호와 당첨 번호를 비교하여 해당 값을 얻는다.
- `int matchingScore` : 일치하는 숫자의 갯수
- `boolean isBonusMatched` : 보너스 번호 일치 여부

그리고 `List<Winning>`에 순회하여 일치하는 당첨기준에 매칭 카운트를 올린다.

`GameRule`의 `matchWinningRule`

```java
/*
	당첨 조건을 점수별로 매핑하여 관리
	해당 점수와 일치하는 당첨 조건이 1개일 경우는 보너스 일치 여부와 상관 없음
	그러나 같은 점수에 보너스 일치 여부에 따라 달라질 경우는 보너스 일치 여부까지 비교
 */
public void matchWinningRule(int matchingScore, boolean isBonusMatched) {
	List<Winning> matchedWinnings = winningRuleSet.get(matchingScore);

	if (matchedWinnings == null) return;

	if (matchedWinnings.size() == 1) {
		Winning matchWinning = matchedWinnings.getFirst();
		matchWinning.countUpIfMatched(matchingScore);*
	}

	if (matchedWinnings.size() > 1) {
		matchedWinnings.forEach(winning -> winning.countUpIfMatched(matchingScore, isBonusMatched));
	}*
}
```

#### 번외 : 여러 요청 데이터 중 잘못된 요청 데이터만 요구
당첨 통계를 산출하기 위해 당첨번호와 보너스 번호를 입력한다.
해당 입력값들을 아래와 같은 DTO에 담아 전달한다.
```java
public record ResultRequest(  
        String winningNumber,  
        String bonusNumber  
) {  
}
```

> *사용자가 잘못된 값을 입력할 경우 `IllegalArgumentException`을 발생*

`winningNumber`는 정상적이나 `bonusNumber`가 적절하지 않은 입력값인 경우에는 어떻게 처리?



# 과제 제출란 소감
## 3주차 과제를 수행하며... 

해당 주차에서는 계층을 세분화하고 책임에 따른 역할 분리에 집중했습니다.
특히 도메인이 더 주도적으로 로직을 수행하도록 지향했습니다.
### 느낀점 3줄요약
1. 머릿속으로 상상하던 추상적 설계는 설계가 아니다!
2. 테스트 코드와 조금은 친해졌다.
3. 메서드가 "한가지 일"을 수행하도록 한다에서 "한가지 일"이란 무엇일까를 생각했다.

### 나의 생각을 글로 표현하기
3주차 문제를 읽고 머릿 속으로 전체적인 구조를 떠올렸습니다.
생각보다 쉬운데? 라는 안일한 생각으로 구현 기능들을 작성하기 시작했습니다.
그러나 막상 글을 작성하기 쉽지 않았습니다.
그 이유는 머릿속에서 상상한 추상적 설계는 전혀 구체적이지 않았기에 글을 작성할 수 없었음을 느꼈습니다.
저의 생각을 점검하고 구체화하기 위해 이번 주차에서는 저의 사고의 흐름을 리드미에 녹여내도록 노력했습니다.

### 테스트 코드와 친해지기
이전과는 달리 테스트 코드를 통해 기능을 검수 후 개발하는 방식으로 진행했습니다.
그 덕분에 테스트 코드 작성에 조금은 익숙해질 수 있었습니다.

특히 `@ValueSource`, `@MethodSource`와 같은 보다 더 편한 테스트 환경을 제공하는 기능을 알게되면서 반복적으로 작업하던 테스트 코드를 줄인 경험이 인상적이었습니다.

다만 1차적으로 기능을 수행 후 리팩토링 과정에서는 테스트를 미비하게 진행한 점은 아쉬운 점으로 남았습니다. 다음번에는 더욱 철저한 테스트 과정을 수행해봄을 다짐했습니다.

### 15줄을 넘기지 말거라...
발행한 로또를 당첨번호와 보너스번호를 비교 후 당첨 기준에 충족되는가를 매칭시키는 메서드를 구현했으나 해당 로직을 수행하는 메서드가 굉장히 길었습니다.

`함수(또는 메서드)의 길이가 15라인을 넘어가지 않도록 구현한다.`를 위반하는 코드였기에 기능을 분리하여 비교 후 나온 "비교 결과값"을 당첨 기준에 매칭하도록 수정했음에도 여전히 15줄을 넘겼습니다. 해당 문제를 해결하기 위해 여러 시도를 수행했습니다. 그 중 `List<당첨기준>`를 `Map<점수,List<당첨기준>>`과 같이 필드의 자료형을 변경하는 시도를 통해 코드의 길이를 줄일 수 있었습니다. 

그러나 해당 자료구조와 메서드의 코드 내용 자체는 직관적이지 않아 더욱 리팩토링이 필요할거 같습니다.

### 다음 목표
- 조금 더 탄탄한 설계 해보기
	- 도메인 및 계층을 세분화
	- 복잡도 고려 및 적절한 자료구조 선택
	- "한가지 일"에 대해 고민
		- 데이터의 타입이 변경될때
	- 그리고 일단 설계시간을 많이 투자!!!
- 통합 테스트 수행
	- 도메인 테스트 코드만을 작성하느 것이 아닌 서비스 계층도 테스트 수행


# 병아리반 포스팅

## 코드리뷰
### 궁금한 점
#### 파싱 과정
1~3주차까지의 많은 결과물들을 보면서 입력값에 대한 파싱 과정을 수행하는 방식이 
설계 의도에 따라 다양하게 설계될 수 있음을 느꼈습니다. 
저는 이번에 `Mapper` 레이어를 추가하였고 `Service`에서 `Mapper`를 호출하고 
`ParseUtil를` 통해 파싱을 거칩니다.
다른 분들은 어떻게 진행하셨는지 궁금합니다!

> 제가 봤던 방식들
> 1. 입력 단계에서 파로 파싱
> 2. Controller에서 파싱
> 3. Domain에서 파싱
> 4. DTO (record) 내부에서 파싱

Controller에서 수행하자
- view는 입력을 받는 것에서 끝남, 비지니스 로직에서는 너무 많은 과정을 수행

- 형식 검증 : 데이터 타입 (Controller에서 수행)
	- notNull, Integer... 

#### 한가지 일이란?
함수의 길이가 15라인 이상이 되지 않도록 설계하라는 요구사항이 3주차에 추가됐습니다.
이를 지키기 위해 함수가 한가지 일에 집중하도록 코드를 작성했습니다.
여러분들이 생각하는 함수가 수행하는 **한가지 일**이란게 무엇일까요?
> 1. 제가 생각하는 프로그래밍에서의 한가지 일은 데이터의 변화가 있을때 라고 생각합니다.
### 나의 아쉬운 점
- 전반적으로 `null` 체킹에 소흘하게 작성한듯함
- 통합 테스트를 수행하지는 못함
- 상속이 다소 적절하지 않게 사용된듯함
### 다음 목표
- 조금 더 탄탄한 설계 해보기
	- 도메인 및 계층을 세분화
	- 복잡도 고려 및 적절한 자료구조 선택
	- "한가지 일"에 대해 고민
		- 데이터의 타입이 변경될때
	- 그리고 일단 설계시간을 많이 투자!!!
- 통합 테스트 수행
	- 도메인 테스트 코드만을 작성하느 것이 아닌 서비스 계층도 테스트 수행

## 발표
### 함수형 인터페이스

> - 사용자가 잘못된 값을 입력할 경우 `IllegalArgumentException`을 발생시키고, "[ERROR]"로 시작하는 에러 메시지를 출력 후 그 부분부터 입력을 다시 받는다.
> 	- `Exception`이 아닌 `IllegalArgumentException`, `IllegalStateException` 등과 같은 명확한 유형을 처리한다.

```java
while (true) {  
    try {  
        반복할 코드
        return;  
    } catch (IllegalArgumentException e) {  
        예외시 행위 
    }  
}
```
반복을 구현하는 방법 중 while try-catch로 구현

> 해당 과제에서는 입력값을 2번 받으므로 위와 같은 똑같은 구조의 코드가 필요함
> 이를 재활용할 수 있는 방법은 없을까?

##### 자바스크립트 예시
```javascript
// 함수를 변수에 선언
const myFunction = () => {
  console.log("안녕");
}

// 함수를 매개변수에 받음
function repeatFive(func) {
  for (let index = 0; index < 5; index++) {
    func();
  }
}

// 이런식으로 실행
repeatFive(myFunction);
```
저는 자바스크립트의 문법이 바로 생각이 났습니다.
자바도 이게 되나?

```java
public void repeat(Object object) {  
    while (true) {  
        try {  
            object(); // Method call expected
            return;  
        } catch (IllegalArgumentException e) {  
            System.out.println(e.getMessage());  
        }  
    }  
}
```
그럴리가 있겠습니까,, 하하,,
> 자바와 자바스크립트는 인도와 인도네시아와 같은 관계

이럴때 사용되는게 **함수형 인터페이스**입니다.

#### 정의 
Functional Interface란  **Object Class의 메서드를 제외**하고 
'구현해야 할 추상 메서드가 하나만 정의된 인터페이스' 를 의미함
> 해당 정의가 와닿지 않다면
> 앞서 제가 언급한 문제상황 같이 함수 자체를 매개변수로 사용하고 싶을때 
> 사용하는 녀석이라고 생각하셔 무리가 없을거 같습니다.
#### 사용시 이점
1. 프로그램 검증이 쉽다
2. 계산한 값을 캐싱 하는 등의 최적화 가능
3. 스레드가 프로그램 상태를 공유하기 때문에 생기는 동시성 문제로부터 자유로운 편
4. 함수 자체를 재사용할 수 있음
#### Consumer
- 매개변수를 받고 리턴값이 없는 친구
- `accept()`로 사용
```java
Consumer<String> shouting = name -> System.out.println(name + "!!!!!");
```
`shouting`에 함수 자체를 저장

```java
shouting.accept("hello world");
// return : hello world!!!!!
```
`accept()`로 호출

```java
public <T> void inputRepeat(Consumer<T> consumer, T params) {  
    while (true) {  
        try {  
            consumer.accept(params);  
            return;  
        } catch (IllegalArgumentException e) {  
            System.out.println(e.getMessage());  
        }  
    }  
}
```
제네릭을 활용한다면 위와 같이 사용가능하다.
`inputRepeat`는 매개변수가 예외가 터지지 않을때까지 해당 매개변수의 반복을 수행합니다

```java
@FunctionalInterface  
interface TripleConsumer<T, U, K> {  
    void accept(T t, U u, K k);  
}
```
매개변수를 3개 이상으로 필요할 경우에는 interface를 따로 선언하여 구현가능하다.

그럼 4개, 5개 필요할때마다 계속 만들어야 할까?

사실 근본적으로 단일 메서드에 3개 이상의 매개변수가 요구된다면
해당 메서드 구조에 대해 다시 한번 생각할 필요가 있다고 생각합니다.
> 메서드가 한가지 일을 수행하는가?

만약 매개값의 수가 다르고 리턴값이 다르다면 별도의 인터페이스를 사용해야 됩니다.
> Supplier Interface : 매개값 X, 리턴값 O
> Function Interface : 매개값 O, 리턴값 O
> Operator Interface : 매개값 O, 리턴값 O (매개값과 리턴값이 동일)
> Predicate Interface : 매개값 O, 리턴값 Boolean
#### 이번과제에 왜 적용을 안했나요
1,2주차와는 다르게 속도감 있는 개발을 하고 싶어서 이번 과제는 일부러 시작을 늦게 했습니다.
그렇다보니 리팩토링 과정에서 아쉬운 점을 발견하고도 수행하지 못했습니다.
> 과제 제출 1시간 전에 함수형 인터페이스에 대한 개념을 발견

이 점은 아쉬움으로 남았습니다.
물론 다른 일 때문에 과제에 투자하는 시간을 줄일 수는 있어도
일부러 줄일 필요는 없다고 생각이 들었습니다.
> 극단적으로 말하면 리팩토링에는 끝이 없기 때문




# 코드리뷰
### 함수 객체에 대해 알아보자
``` java
public static <T> T readWithRetry(Supplier<T> supplier) {
	while (true) {
		try {
			return supplier.get();
		} catch (IllegalArgumentException | IllegalStateException e) {
			System.out.println(ERROR_PREFIX + e.getMessage());
		} catch (Exception e) {
			System.out.println(ERROR_PREFIX + "알 수 없는 오류가 발생했습니다.");
			throw e;
		}
	}
}
```

### ENUM 활용
```java
package lotto.domain;

public enum LottoRank {

    FIFTH(3, false, 5_000),
    FOURTH(4, false, 50_000),
    THIRD(5, false, 1_500_000),
    SECOND(5, true, 30_000_000),
    FIRST(6, false, 2_000_000_000)
    ;

    private final int matchCount;
    private final boolean hasBonus;
    private final int prize;

    LottoRank(int matchCount, boolean hasBonus, int prize) {
        this.matchCount = matchCount;
        this.hasBonus = hasBonus;
        this.prize = prize;
    }

    public static LottoRank from(int matchCount, boolean bonusMatch) {
        for (LottoRank rank : values()) {
            if (rank.matchCount == matchCount && rank.hasBonus == bonusMatch) {
                return rank;
            }
        }
        return null;
    }

    public int getMatchCount() {
        return matchCount;
    }

    public boolean hasBonus() {
        return hasBonus;
    }

    public int getPrize() {
        return prize;
    }
}
```


```java
private <T> T getInput(Supplier<T> supplier) {
	while (true) {
		try {
			return supplier.get();
		} catch (IllegalArgumentException e) {
			errorView.printError(e.getMessage());
		}
	}
}

private <I, O> O getInput(Function<I, O> function, I param) {
	while (true) {
		try {
			return function.apply(param);
		} catch (IllegalArgumentException e) {
			errorView.printError(e.getMessage());
		}
	}
}
```