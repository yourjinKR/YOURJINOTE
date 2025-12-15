
# TimeStamp

Timestamp란 UTC 기준으로 특정 순간을 나타내는 값을 의미한다.

일반적으로 다음 특징을 가진다.

- 기준 시점: `1970-01-01T00:00:00Z (Epoch)`
- 표현 방식:
    - 초(second)
    - 밀리초(millisecond)
    - 나노초(nanosecond)
- 전 세계 어디서나 **같은 순간을 동일하게 가리킴**

```
1734220800000  ← timestamp (milliseconds)
```

## 언제 사용하는가?

시간 차이 연산에 유리하므로 로깅 과정에서 시간을 다룰 때 주로 사용한다. (ex. 응답 시간...)

# Java에서 Timestamp

### Instant

`java.time` 패키지에서는 `Timestamp`를 별개로 지원하지 않는다.  
Timestamp 개념을 `Instant`가 담당한다.

```java
Instant now = Instant.now();
System.out.println(now);
// 2025-12-15T01:12:30.123456Z
```

> Z는 UTC(Zero offset)를 의미한다.

### java.sql.Timestamp

```
java.sql.Timestamp
```

- JDBC 호환용
- Java 8 이전 API
- **새 코드에서는 비권장**