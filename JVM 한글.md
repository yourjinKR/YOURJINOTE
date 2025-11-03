```
-Dfile.encoding="UTF-8" -Dsun.stderr.encoding="UTF-8" -Dsun.stdout.encoding="UTF-8"
```


```

but could not find:
  ["6개 일치 (2,000,000,000원) - 0개", "총 수익률은 62.5%입니다."]

```


레이싱 경주와 같은 경우는 기존 필드를 랜덤하게 변경하는 과정이지만

로또 게임은 객체의 필드값의 조건 자체가 랜덤이다.
또한 로또 게임에서 필드값을 더 만들 수 없음.
이 과정에서 로또 객체 내부에서 검증 과정을 어떻게 구현해야 될까?
필드값을 





```java
package lotto.domain;  
  
import java.util.ArrayList;  
import java.util.List;  
import java.util.function.Predicate;  
import lotto.util.ErrorMessage;  
  
public class Lotto {  
    private final List<Integer> numbers;  
  
    public Lotto(LottoPickRule pickRule) {  
        List<Integer> numbers = pickRule.pick();  
          
        validate(numbers, pickRule);  
        this.numbers = numbers;  
    }  
  
    private void validate(List<Integer> numbers, LottoPickRule pickRule) {  
        validateSize(numbers, pickRule);  
        validateDuplicate(numbers, pickRule);  
        validateRange(numbers, pickRule);  
    }  
  
    private void validateSize(List<Integer> numbers, LottoPickRule pickRule) {  
        if (numbers.size() != pickRule.getCount()) {  
            throw new IllegalArgumentException(ErrorMessage.UNMATCH_WINNING_AMOUNT.getMessage());  
        }  
    }  
  
    private void validateDuplicate(List<Integer> numbers, LottoPickRule pickRule) {  
        long numberUniqueSize = numbers.stream().distinct().count();  
  
        if (numberUniqueSize != pickRule.getCount()) {  
            throw new IllegalArgumentException(ErrorMessage.DUPLICATE_LOTTO_NUMBER.getMessage());  
        }  
    }  
  
    private void validateRange(List<Integer> numbers, LottoPickRule pickRule) {  
        boolean outOfRange = numbers.stream()  
                .anyMatch(number -> (number < pickRule.getStartInclusive()) || (number > pickRule.getEndInclusive()));  
  
        if (outOfRange)  
            throw new IllegalArgumentException(ErrorMessage.OUT_OF_RANGE_NUMBER.getMessage());  
    }  
  
    public int getMatchingScore(WinningLotto winningLotto) {  
        List<Integer> winningNumbers = winningLotto.getNumbers();  
  
        List<Integer> matchList = numbers.stream()  
                .filter(number -> winningNumbers.stream()  
                        .anyMatch(Predicate.isEqual(number)))  
                .toList();  
  
        return matchList.size();  
    }  
  
    public boolean isBonusMatched(WinningLotto winningLotto) {  
        int bonusNumber = winningLotto.getBonusNumber();  
        return numbers.contains(bonusNumber);  
    }  
  
    public List<Integer> getNumbers() {  
        return new ArrayList<>(numbers);  
    }  
}
```