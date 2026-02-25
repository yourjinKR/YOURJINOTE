# SecurityConfig

## 인가

권한별로 허용할 요청들을 다음과 같이 구조화시킬 수 있다.  

```java
http.authorizeHttpRequests(auth -> auth  
        .requestMatchers("/jwt/exchange", "/jwt/refresh", "/error").permitAll()  
        .requestMatchers(HttpMethod.POST, "/api/user").permitAll()  
        .requestMatchers(HttpMethod.GET, "/api/check", "/api/user/exist/**").permitAll()  
        .requestMatchers(HttpMethod.GET, "/api/user/me").hasAnyAuthority("MEMBER", "TRAINER", "ADMIN")  
        .anyRequest().authenticated()  
);
```

만약 PathVariable를 사용할 경우 경로를 `"/api/user/exist/**"`와 같이 명시해야 한다.  