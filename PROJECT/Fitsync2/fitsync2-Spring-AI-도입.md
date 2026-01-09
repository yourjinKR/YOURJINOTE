
## Structured Output 도입

자바에 맞게 구조화된 응답으로 값을 받기 위해 위와 같이 `entity()`를 통해 구현

```java
return chatClient.prompt(prompt)  
        .call()  
        .entity(AIRoutineResponse.class);
```

그러나 반환 타입이 `List<AIRoutineResponse>`로 변경하게 됐다.

```java
return chatClient.prompt(prompt)  
        .call()  
        .entity(new ParameterizedTypeReference<>() {});
```

`ParameterizedTypeReference` 를 활용


```java
ChatResponse response = chatClient.prompt(prompt)  
        .tools(new AITools(exerciseRepository, exerciseMapper))  
        .call()  
        .chatResponse();
```