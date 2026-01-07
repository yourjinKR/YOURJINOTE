
## Intellij에서 관리

![Pasted image 20260107175824](../../GALLERY/Pasted%20image%2020260107175824.png)


## OS 수준에서 관리

### Powershell

```shell
setx OPEN_AI_SECRET_KEY "sk-xxxxx"
```

설정 후 shell 재실행하여 아래 명령어로 확인

```shell
echo $env:OPEN_AI_SECRET_KEY
```

이후 인텔리제이와 같은 IDE를 재실행

## .env 파일

### .env

```
OPEN_AI_SECRET_KEY=12312312
```

### .application

```properties
spring.config.import=optional:file:.env[.properties]  
spring.ai.openai.api-key=${OPEN_AI_SECRET_KEY}
```

`.env` 파일을 통해 환경변수를 설정했다면 `spring.config.import`를 사용하여 프로퍼티들을 불러와야 한다.

### .yml

```yml
spring:
  config:
    import: optional:file:.env[.properties]
```

