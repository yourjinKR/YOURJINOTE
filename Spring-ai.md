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
    api-key: ${OPEN_AI_SECRET_KEY}
```

환경변수 설정 후 실행