
# CAS

CAS(Compare-And-Swap)은 메모리의 현재 값이 예상 값과 같을 떄만 새로운 값으로 교체하는 [원자적](RESOURCE/CS/원자성.md) 연산이다.

## 핵심

> CAS는 “대기” 대신 “재시도”를 선택한 방식

### 기존 락 기반 동기화의 비용

synchronized / Lock 사용 시 내부에서 아래와 같은 작업을 수행한다.

- 락 획득 경쟁
- BLOCKED 상태 진입
- 컨텍스트 스위칭
- 스레드 재스케줄링

스레드가 많아질수록 성능이 급격히 감소한다.

### CAS 연산 구조

```java
CAS(메모리 주소, 기대값, 새 값)

if (메모리 값 == 기대값) {
    메모리 = 새 값
    성공
} else {
    실패
}
```

해당 전체 구조가 CPU 한 번의 원자 연산으로 수행된다.

#### CAS 기반 증가 과정 예시

초기값: `count = 0`

스레드 A, B 동시에 증가:
1. A 읽음 → 0
2. B 읽음 → 0
    

A :
- CAS(0, 1) → 성공 → count = 1
    

B :
- CAS(0, 1) → 실패 (이미 1)
- 다시 읽음 → 1
- CAS(1, 2) → 성공 → count = 2
    

👉 락 없이도 **정확한 결과**

<br>

# 출처

[baeldung - ABA Problem](https://www.baeldung.com/cs/aba-concurrency)