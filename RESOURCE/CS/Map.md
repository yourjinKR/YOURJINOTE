# Map

- key-value pair을 저장하는 추상 자료형
- 같은 key를 가진 pair는 최대 한 개만 존재
- associative array, dictionary라고도 부른다

Map의 구현체는 크게 2가지로 분류한다.

- Hash Table
- Tree-based

## Hash Table

- 배열과 [해시 함수](Hash-Function-해시-함수.md)를 사용하여 Map을 구현한 자료 구조
- 상수 시간으로 데이터에 접근

### Hash table resizing

