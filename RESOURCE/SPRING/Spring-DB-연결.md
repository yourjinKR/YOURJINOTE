## MySQL

스키마와 계정 생성 후 아래와 같이 입력

### build.gradle

```groovy
runtimeOnly 'com.mysql:mysql-connector-j'
```

### properties

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.datasource.url=jdbc:mysql://localhost:3306/${DATABASE_NAME}?serverTimezone=Asia/Seoul

spring.datasource.username=${MYSQL_USERNAME}
spring.datasource.password=${MYSQL_PASSWORD}

```

### yaml

```yaml
spring:
  datasource:
    # MySQL
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/${DATABASE_NAME}?&serverTimezone=Asia/Seoul
    username: ${MYSQL_USERNAME}
    password: ${MYSQL_PASSWORD}
```

[MySql-schema](../DB/MySql/MySql-schema.md)

[출처](https://chaeyami.tistory.com/244#google_vignette)

## Postgresql


## Oracle