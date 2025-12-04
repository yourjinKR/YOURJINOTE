# @Enumerated
## 컬럼에 enum 적용하기
```java
@Column(name = "social_provider")  
@Enumerated(EnumType.STRING)  
private SocialProvider socialProvide;
```
