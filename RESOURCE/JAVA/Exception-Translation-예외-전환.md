# 예외 전환

예외 전환이란 상위 계층이 하위 계층에서 발생한 특정 예외를 그대로 노출하지 않고, 상위 계층의 의미에 맞는 예외로 바꾸어 다시 던지는 것을 말한다.

## 예제

```java
public void registerUser(User user) {
    try {
        userRepository.save(user);
    } catch (SQLException e) {
        // SQLException(하위 계층 예외)을 
        // DuplicateUserIdException(상위 계층/비즈니스 예외)으로 전환
        throw new DuplicateUserIdException("이미 존재하는 아이디입니다.", e);
    }
}
```

# 출처

[gemini chat](https://gemini.google.com/share/6657a66d0fa3)  
