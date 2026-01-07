# 프로젝트 세팅

### build.gradle

```groovy
ext {  
    set('springAiVersion', "1.1.2")  
}
```

```groovy
dependencies {  
    // Spring AI  
    implementation 'org.springframework.ai:spring-ai-starter-model-openai'  
}
```

```groovy
dependencyManagement {  
    imports {  
        mavenBom "org.springframework.ai:spring-ai-bom:${springAiVersion}"  
    }  
}
```

### application.yaml

```yaml
ai:  
  openai:  
    api-key: ${OPEN_AI_S ECRET_KEY}
```

환경변수 설정 후 실행


# Structured Output

### DTO

```java
return chatClient.prompt(prompt)  
        .call()  
        .entity(AIRoutineResponse.class);
```

### 제네릭 타입

```java
return chatClient.prompt(prompt)  
        .call()  
        .entity(new ParameterizedTypeReference<>() {});
```

`List<DTO>`와 같이 제네릭을 사용하는 반환값이라면 위와 같이 `ParameterizedTypeReference`를 작성한다.