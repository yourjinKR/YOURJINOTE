

# 개인 프로젝트
### 프로필 상세조회 응답 DTO


### 기존 조회 방식

- 유저 ID를 기반으로 검색하여 프로필 엔티티를 불러옴

```java
@Override  
public UserProfileDetailResponse view(long userId) {  
  
    UserProfile profile = userProfileRepository.findByUserId(userId)  
            .orElseThrow(() -> new RestApiException(UserProfileException.NOT_FOUND, userId));  
  
    return userProfileMapper.toDto(profile);  
}
```

### 응답 DTO

```java
public record UserProfileDetailResponse(  
        long id,  
        User user,  
        Set<WorkoutGoal> workoutGoals,  
        Set<ExerciseCategory> exerciseCategories,  
        String disease,  
        Double height,  
        Double weight,  
        Double skeletalMuscleMass,  
        Double bodyFatMass,  
        Double bodyFatPercentage,  
        Double bmi  
) {  
  
    public record User(  
            String name,  
            Long age,  
            boolean hidden  
    ) {  
  
    }  
}
```

- 프로필에 대한 상세정보가 메인
- 유저 정보는 내부 레코드에서 관리


#### 1차 리팩토링

```java
public record UserProfileDetailResponse(  
        long id,  
          
        String name,  
        Long age,  
        boolean hidden,  
  
        Set<WorkoutGoal> workoutGoals,  
        Set<ExerciseCategory> exerciseCategories,  
        String disease,  
        Double height,  
        Double weight,  
        Double skeletalMuscleMass,  
        Double bodyFatMass,  
        Double bodyFatPercentage,  
        Double bmi  
) {  
}
```

한 객체의 너무 많은 필드가 존재함.

`id`가 `User` 혹은 `UserProfile`의 PK인지 구분하기 어렵다.

> 해당 `id`는 `UserProfile`의 `pk`이다.

#### 2차 리팩토링

```java
public record UserProfileDetailResponse(  
        Set<WorkoutGoal> workoutGoals,  
        Set<ExerciseCategory> exerciseCategories,  
        String disease,  
        Double height,  
        Double weight,  
        Double skeletalMuscleMass,  
        Double bodyFatMass,  
        Double bodyFatPercentage,  
        Double bmi  
) {  
}
```

```java
public record UserHeaderInfoResponse(  
        String name,  
        Long age,  
        boolean hidden  
) {  
}
```

- 위와 같이 엔티티 별로 계층을 구분하여 관리
- 두 엔티티의 pk 중 어떤 것을 사용할지에 대해 고민함

```java
public record UserWithProfileResponse(  
        long userId,  
        UserHeaderInfoResponse user,  
        long profileId,  
        UserProfileDetailResponse profile  
) {  
}
```

- 결국 아래와 같이 각각의 엔티티의 pk를 사용하기 위해 위와 같이 구분



## 시스템 메세지 테스트

유저와 유저프로필에 대한 응답 DTO를 재사용성 있게 리팩토링한 덕분에 프롬프트 요청 DTO를 설계하는데 용이함을 느꼈다.

### 프롬프트

AI 루틴 추천 API를 구현했고 POSTMAN에서 테스트를 해야 함. 아래와 같이 응답 DTO를 구현했으니 해당 구조에 맞는 json 데이터 만들어줘.

```java
/**  
 * AI 운동 루틴 추천 서비스 요청<br>  
 * - 사용자 정보  
 */public record AIRoutineRequest(  
        UserInfo userInfo  
) {  
    public record UserInfo(  
            UserHeaderInfoResponse userHeaderInfo,  
            UserProfileDetailResponse userDetailInfo  
    ) {  
  
    }  
}

```

```java
package app.fitsync.domain.user.dto;  
  
public record UserHeaderInfoResponse(  
        String name,  
        Long age,  
        boolean hidden  
) {  
}
```

```java
package app.fitsync.domain.profile.dto;  
  
import app.fitsync.domain.exercise.entity.ExerciseCategory;  
import app.fitsync.domain.profile.entity.WorkoutGoal;  
import java.util.Set;  
  
public record UserProfileDetailResponse(  
        Set<WorkoutGoal> workoutGoals,  
        Set<ExerciseCategory> exerciseCategories,  
        String disease,  
        Double height,  
        Double weight,  
        Double skeletalMuscleMass,  
        Double bodyFatMass,  
        Double bodyFatPercentage,  
        Double bmi  
) {  
}
```

### 결과

![Pasted image 20260106194548](GALLERY/Pasted%20image%2020260106194548.png)

```java
@Override  
public String generateRoutine(AIRoutineRequest request) {  
  
    ChatClient chatClient = ChatClient.create(openAiChatModel);  
  
    SystemMessage systemMessage = new SystemMessage(SystemPromptConstant.ROUTINE_REQUEST);  
    UserMessage userMessage = new UserMessage(MessageFormat.format("나에 대한 정보를 보여줄테니 운동 루틴 추천해줘. 내 정보 : {0}", request.toString()));  
    AssistantMessage assistantMessage = new AssistantMessage("");  
  
    OpenAiChatOptions options = OpenAiChatOptions.builder()  
            .model(MODEL)  
            .temperature(0.7)  
            .build();  
  
    Prompt prompt = new Prompt(List.of(systemMessage, userMessage, assistantMessage), options);  
  
    return chatClient.prompt(prompt)  
            .call()  
            .content();  
}
```

단순히 요청 레코드를 `toString`하여 프롬프트로 전달함.
