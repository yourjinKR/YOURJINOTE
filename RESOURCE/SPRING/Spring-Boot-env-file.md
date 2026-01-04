
```
OPEN_AI_SECRET_KEY=12312312
```

```
spring.config.import=optional:file:.env[.properties]  
spring.ai.openai.api-key=${OPEN_AI_SECRET_KEY}
```

`.env` 파일을 통해 환경변수를 설정했다면 `spring.config.import`를 사용하여 프로퍼티들을 불러와야 한다.