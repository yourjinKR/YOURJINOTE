
# 통합

```properties
# OAuth2 : NAVER registration
spring.security.oauth2.client.registration.naver.client-name=naver
spring.security.oauth2.client.registration.naver.client-id=
spring.security.oauth2.client.registration.naver.client-secret=
spring.security.oauth2.client.registration.naver.redirect-uri=http://localhost:8080/login/oauth2/code/naver
spring.security.oauth2.client.registration.naver.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.naver.scope=name,email
spring.security.oauth2.client.provider.naver.authorization-uri=https://nid.naver.com/oauth2.0/authorize
spring.security.oauth2.client.provider.naver.token-uri=https://nid.naver.com/oauth2.0/token
spring.security.oauth2.client.provider.naver.user-info-uri=https://openapi.naver.com/v1/nid/me
spring.security.oauth2.client.provider.naver.user-name-attribute=response

# OAuth2 : Google registration
spring.security.oauth2.client.registration.google.client-name=google
spring.security.oauth2.client.registration.google.client-id=
spring.security.oauth2.client.registration.google.client-secret=
spring.security.oauth2.client.registration.google.redirect-uri=http://localhost:8080/login/oauth2/code/google
spring.security.oauth2.client.registration.google.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.google.scope=profile,email
```

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          naver:
            client-name: naver
            client-id: 
            client-secret: 
            redirect-uri: http://localhost:8080/login/oauth2/code/naver
            authorization-grant-type: authorization_code
            scope:
              - name
              - email

          google:
            client-name: google
            client-id: 
            client-secret: 
            redirect-uri: http://localhost:8080/login/oauth2/code/google
            authorization-grant-type: authorization_code
            scope:
              - profile
              - email

        provider:
          naver:
            authorization-uri: https://nid.naver.com/oauth2.0/authorize
            token-uri: https://nid.naver.com/oauth2.0/token
            user-info-uri: https://openapi.naver.com/v1/nid/me
            user-name-attribute: response

```


# NAVER

```properties
# OAuth2 : NAVER registration
spring.security.oauth2.client.registration.naver.client-name=naver
spring.security.oauth2.client.registration.naver.client-id=${NAVER_CLIENT_ID}  
spring.security.oauth2.client.registration.naver.client-secret=${NAVER_SECRET_KEY}
spring.security.oauth2.client.registration.naver.redirect-uri=http://localhost:8080/login/oauth2/code/naver
spring.security.oauth2.client.registration.naver.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.naver.scope=name,email
spring.security.oauth2.client.provider.naver.authorization-uri=https://nid.naver.com/oauth2.0/authorize
spring.security.oauth2.client.provider.naver.token-uri=https://nid.naver.com/oauth2.0/token
spring.security.oauth2.client.provider.naver.user-info-uri=https://openapi.naver.com/v1/nid/me
spring.security.oauth2.client.provider.naver.user-name-attribute=response
```

```yaml
security:  
  oauth2:  
    client:  
      registration:  
        naver:  
          client-name: naver  
          client-id: ${NAVER_CLIENT_ID}  
          client-secret: ${NAVER_SECRET_KEY}  
          redirect-uri: http://localhost:8080/login/oauth2/code/naver  
          authorization-grant-type: authorization_code  
          scope:  
            - name  
            - email  
            - gender  
            - birthday  
            - birthyear  
      provider:  
        naver:  
          authorization-uri: https://nid.naver.com/oauth2.0/authorize  
          token-uri: https://nid.naver.com/oauth2.0/token  
          user-info-uri: https://openapi.naver.com/v1/nid/me  
          user-name-attribute: response
```

[참고](https://adjh54.tistory.com/591)

![Pasted image 20260129130818](GALLERY/Pasted%20image%2020260129130818.png)

# KAKAO

## 응답 형태

```json
{
   "id" : 4440509450,
   "connected_at" : "2025-09-10T13:13:57Z",
   "kakao_account" : {
      "name_needs_agreement" : false,
      "name" : "유어진",
      "has_email" : true,
      "email_needs_agreement" : false,
      "is_email_valid" : true,
      "is_email_verified" : true,
      "email" : "young680415@naver.com"
   }
}
```

[참고](https://adjh54.tistory.com/591)