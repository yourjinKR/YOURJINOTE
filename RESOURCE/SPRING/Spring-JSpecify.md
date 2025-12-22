# 기존 Non-Null 처리 방식

![Pasted image 20251222201126](../../GALLERY/Pasted%20image%2020251222201126.png)

Non-Null한 구조임에도 불구하고 IDE에서는 null에 대한 체크를 명시하도록 요구한다.
그렇다면 지금과 같이 모든 인자에 `@NonNull`를 명시해야 되는 불편함이 있다.

```java
@PostMapping("/api/exercise")  
public ResponseEntity<@NonNull ExerciseResponse> createExercise(@Valid @RequestBody ExerciseRequest request) {  
  
    ExerciseResponse response = exerciseService.createExercise(request);  
    return ResponseEntity.ok(response);  
}
```

이를 해결하기 위한 방법은 다음과 같지만 문제점이 존재한다.

### `@NonNullApi`

```java
@org.springframework.lang.NonNullApi
package app.fitsync.domain.routine.controller;
```

패키지에 `package-info.java` 파일을 추가하여 위와 같이 작성한다면 해당 패키지 내부 타입에 대한 Non-Null 처리를 해결한다.

```java
@Target(ElementType.PACKAGE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Nonnull  
@TypeQualifierDefault({ElementType.METHOD, ElementType.PARAMETER})  
@Deprecated(since = "7.0")  
public @interface NonNullApi {  
}
```

그러나 해당 방식은 더 이상 지원되지 않음

### `Objects.requireNonNull()`

```java
@PostMapping("/api/routine")
public ResponseEntity<RoutineResponse> create(@Valid @RequestBody RoutineRequest request) {
    RoutineResponse response = routineService.createRoutine(request);
    
    // JDK 표준 메서드로 null이 아님을 확정 (IDE가 이 지점 이후부터는 Non-null로 인식)
    return ResponseEntity.ok(java.util.Objects.requireNonNull(response));
}
```

그러나 해당 경고는 컴파일 시점에 대한 것이기 때문에 여전히 문제 발생

# JSpecify 사용법

이는 JSpecify의 `@NullMarked`를 통해 해결할 수 있다.

```
implementation 'org.jspecify:jspecify:1.0.0'
```

```java
@RestController  
@RequiredArgsConstructor  
@NullMarked  // 추가!!!
public class RoutineController {  
  
    private final RoutineServiceInterface routineService;  
  
    @PostMapping("/api/routine")  
    public ResponseEntity<RoutineResponse> create(@Valid @RequestBody RoutineRequest request) {  
  
    }  
  
    @GetMapping("/api/routines")  
    public ResponseEntity<Page<RoutineListResponse>> getRoutineList(  
            @RequestParam(required = false) Long writerId,  
            @RequestParam(required = false) Long ownerId,  
            @PageableDefault(size = 5, sort = "id", direction = Direction.DESC) Pageable pageable  
    ) {  
  
    }  
  
    @GetMapping("/api/routine/{routineId}")  
    public ResponseEntity<RoutineDetailResponse> getRoutineList(@PathVariable long routineId) {  
  
    }  
}
```


# 출처

[baeldung - package-info.java 파일](https://www.baeldung.com/java-package-info)  
[[Java] Objects.requireNonNull 은 왜 사용할까?](https://hudi.blog/java-requirenonnull/)  
[Spring 7.0 부터 도입되는 Jspecify에 대해 알아보기!](https://seungpnag.tistory.com/103)  
[baeldung - JSpecify를 사용한 Java의 널 안전성 실무 가이드](https://www.baeldung.com/java-jspecify-null-safety)
