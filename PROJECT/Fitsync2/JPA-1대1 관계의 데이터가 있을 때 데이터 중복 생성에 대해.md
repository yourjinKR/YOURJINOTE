
# 상황

```java
class User {} : class UserProfile {}
```

유저와 유저 프로필이 1대1 관계이다.

한 유저가 여러개의 프로필을 생성할 수 없다.

이는 이미 DDL 때문에 `DataIntegrityViolationException`이 발생한다.

```json
{
    "code": "INTERNAL_SERVER_ERROR",
    "message": "Internal server error"
}
```

```
could not execute statement [Duplicate entry '10' for key 'user_profiles.UKe5h89rk3ijvdmaiig4srogdc6']
```

해당 예외처리에 대한 추가적인 작업은 필요 없을까?

내가 생각한 방향성은 아래와 같다

- `try-catch` 블럭을 사용
- 내부적으로 유저 프로필 검색 후 데이터가 있을 때 예외 발생



### try-catch


```java
@Override  
@Transactional  
public UserProfileResponse create(UserProfileRequest request) {  
  
    long userId = request.userId();  
    User user = userRepository.findById(userId)  
            .orElseThrow(IllegalArgumentException::new);  
  
    UserProfile profile = userProfileMapper.toEntity(request, user);  
  
    try {  
        UserProfile save = userProfileRepository.save(profile);  
        return new UserProfileResponse(save.getId());  
    } catch (DataIntegrityViolationException e) {  
        throw new RestApiException(UserProfileException.DUPLICATE, userId);  
    }  
}
```

```json
{
    "code": "DUPLICATE",
    "message": "이미 해당 유저의 프로필이 존재합니다 ID : 10"
}
```

위와 같이 구현함.

그런데 고민점이 있다, 예외처리의 이점 중 하나는 "예외를 일찍 인지하여 추후 해도 되지 않을 작업을 수행하지 않는다." 라는 점이다.

그런 측면에서는 위와 같은 구조는 리팩토링 전후로 얻는 이점은 아니었다.

어떻게 할까..?

### 리팩토링

- 유저 id로 유저가 있는지 확인 -> 이거 대신에 유저의 id로 유저의 프로필이 있는지 확인

```java
@Override  
@Transactional  
public UserProfileResponse create(UserProfileRequest request) {  
  
    long userId = request.userId();  
    boolean profilePresent = userProfileRepository.findByUserId(userId).isPresent();  
  
    if (profilePresent)  
        throw new RestApiException(UserProfileException.DUPLICATE, userId);  
  
    User user = userRepository.findById(userId)  
            .orElseThrow(() -> new RestApiException(UserException.NOT_FOUND));  
  
    UserProfile profile = userProfileMapper.toEntity(request, user);  
  
    UserProfile save = userProfileRepository.save(profile);  
    return new UserProfileResponse(save.getId());  
}

```

- 특정 userId의 프로필이 있는지 확인, 있다면 throw
- 특정 userId의 유저가 있는지 확인, 없다면 throw
- 유저 프로필 생성

> userId로 
