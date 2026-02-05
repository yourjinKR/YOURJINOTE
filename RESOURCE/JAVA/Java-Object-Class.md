# Object 클래스

자바의 모든 클래스는 Object의 파생 형식이다.

## 메서드

- `equals()` : 매개변수로 전달된 참조자와 `this`가 가리키는 대상이 같은 값인지 비교
- `hashCode()` : 객체 식별을 위한 고유 해시결과 값을 반환, 해당 반환값은 Unique
- `toString()` : `클래스명@해시코드` 형식의 문자열을 반환
- `getClass()`

### equals() 재정의

특정 객체를 여러 참조자를 갖지 않는 이상 비교인자는 무조건 `false`이다.<br>그러나 [동일성이 아닌 동등성에 대한 비교](../CS/동등성과%20동일성.md)를 하기 위해서 주로 재정의 과정을 거친다.

# 출처
[인프런 강의 - 널널한 개발자](https://www.inflearn.com/course/%EB%8F%85%ED%95%98%EA%B2%8C-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-java-part2/dashboard)