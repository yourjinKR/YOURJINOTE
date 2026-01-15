# SOLID

함수와 데이터 구조를 효과적으로 결합하고, 이 집합들을 서로 유기적으로 연결하는 방법을 제시한다.

- S : Single Responsibility Principle (단일 책임 원칙)
- O : Open-Closed Principle (개방-폐쇄 원칙)
- L : Liskov Substitution Principle (리스코프 치환 원칙)
- I : Interface Segregation Principle(인터페이스 분리 원칙)
- D : Dependency Inversion Principle(의존성 역전 원칙)

SOLID 원칙의 핵심은 변경에 유연하고 이해하기 쉬운 소프트웨어 구조를 만드는데 있다.

1. 변경에 유연한 구조  
2. 이해하기 쉬운 구조

> 변경에 유연하다라는 말은 결합도는 낮고 응집도는 높은 구조를 의미한다.  

## Single Responsibility Principle (단일 책임 원칙)

단일 책임 원칙이란 하나의 클래스가 하나의 책임만을 가져야 한다는 것을 의미한다.

해당 원칙을 준수할 시 다음과 같은 이점을 가질 수 있다.

- 테스트 : 책임이 하나만 있는 클래스는 테스트 케이스 수가 훨씬 적어진다
- 결합도가 낮을수록 단일 클래스에 포함된 기능이 적어지고 의존성 또한 적어진다.
- 구성 : 규모가 작고 잘 정리된 클래스는 획일적인 클래스보다 검색하기 쉽다

## Open-Closed Principle(개방-폐쇄 원칙)

개방 폐쇄 원칙이란 클래스 확장에 대해서는 개방적이어야 하지만 수정에 대해서는 폐쇄적이어야 한다.

<br>

# 출처

[카카오 뱅크 기술 블로그](https://tech.kakaobank.com/posts/2411-solid-truth-or-myths-for-developers/)  
