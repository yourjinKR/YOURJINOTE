#test-code
### 빌드
```groovy
testImplementation group: 'org.assertj', name: 'assertj-core', version: '3.21.0'
```

### static import
```java
import static org.assertj.core.api.Assertions.*;
```

### @BeforeEach
- 각각의 테스트를 진행할때마다 특정 로직을 반복
```java
private Set<Integer> numbers;
@BeforeEach
void setUp() {
    numbers = new HashSet<>();
    numbers.add(1);
    numbers.add(1);
    numbers.add(2);
    numbers.add(3);
    numbers.add(4);
}
```

## 기본적인 검증
### @Test
- 해당 코드가 테스트임을 명시

### @DisplayName
- 해당 테스트의 이름 표시 가능
```groovy
@Test
@DisplayName("set 크기 확인")
void checkSize() {
    assertThat(numbers.size())
            .isEqualTo(4);
}
```

## 여러 값들을 한번에 검증
### @ParameterizedTest
- 해당 코드는 많은 값들을 필요하는 테스트임을 명시

### @ValueSource
- 테스트에 주입할 값을 어노테이션에 배열로 지정
- 타입 명시 및 배열 내부에 값을 넣음
```java
@ParameterizedTest
@DisplayName("set 값 있는지 확인")
@ValueSource(ints = {1,2,3,4})
void contains(int value) {
    assertThat(numbers.contains(value))
            .isTrue();
}

@ParameterizedTest
@DisplayName("set에 없는 값 확인")
@ValueSource(ints = {5,6,7,8})
void containsInvalidValue(int value) {
    assertThat(numbers.contains(value))
            .isFalse();
}
```

### @MethodSource
```java
@ParameterizedTest
@MethodSource("generateData")
@DisplayName("WinningLotto 또한 Lotto와 같은 검증을 거침")
void validateLottoTest(List<Integer> numbers, int bonusNumber) {
    assertThatThrownBy(() -> new WinningLotto(numbers, bonusNumber))
            .isInstanceOf(IllegalArgumentException.class);
}

static Stream<Arguments> generateData() {
    return Stream.of(
            Arguments.of(Arrays.asList(1,2,3,4,5), 6),
            Arguments.of(Arrays.asList(1,2,3,4,5,5), 6),
            Arguments.of(Arrays.asList(1,2,3,4,5,100), 6)
    );
}
```