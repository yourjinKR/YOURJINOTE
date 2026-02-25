
프로젝트의 API를 문서화 한다면 서버와 클라이언트 협업 간 이점이 있다.  
이러한 API 명세서에 대한 표준 중 OpenAPI를 Spring docs와 Swagger UI를 통해 간편히 구현 가능하다.

OAS(OpenAI)는 json/yaml 형식이 있다. (이는 gpt에게 맡기자)  
OAS를 Swagger UI에 넣으면 특정 경로에 명세서를 띄울 수 있다.

# OpenApiConfig

기본설정값

```java
@Configuration
@OpenAPIDefinition(
        info = @Info(
                title = "핏싱크2 백엔드 API",
                version = "v1",
                description = "Spring 애노테이션과 엔드포인트 시그니처 기반 자동 생성 OpenAPI 문서입니다."
        ),
        security = @SecurityRequirement(name = "bearerAuth")
)
@SecurityScheme(
        name = "bearerAuth",
        type = SecuritySchemeType.HTTP,
        scheme = "bearer",
        bearerFormat = "JWT",
        in = SecuritySchemeIn.HEADER
)
public class OpenApiConfig { }
```

## JWT 설정

프로젝트에 JWT가 적용되어 있는데

아래와 같이 간단한 설정만으로도 끝나는건가?

```java
@SecurityScheme(
        name = "bearerAuth",
        type = SecuritySchemeType.HTTP,
        scheme = "bearer",
        bearerFormat = "JWT",
        in = SecuritySchemeIn.HEADER
)
```

문서에 기본적으로 보안이 요구됨을 명시하기 위해 아래와 같은 코드로 전역에 제한을 걸 수 있다.

```java
@OpenAPIDefinition(  
security = @SecurityRequirement(name = "bearerAuth")  
)
```

> 위 애노테이션들은 **OpenAPI 문서/Swagger UI 동작을 위한 것**일뿐 실질적인 보안이 이뤄지는 것은 X

```java
@Operation(summary = "로그인")  
@SecurityRequirement(name = "")
```

로그인과 같은 작업은 인증이 필요 없기에 위와 같은 추가 작업 가능


## 그룹화

해당 포맷에서 버전 그룹화를 하려면 어떻게 하는가?

```java
@Bean
public GroupedOpenApi v1Api() {
	return GroupedOpenApi.builder()
			.group("v1")
			.pathsToMatch("/api/v1/**") // v1 경로 규칙
			.build();
}

@Bean
public GroupedOpenApi v2Api() {
	return GroupedOpenApi.builder()
			.group("v2")
			.pathsToMatch("/api/v2/**") // v2 경로 규칙
			.build();
}
```

위와 같이 Bean 등록을 통해 API 버전별 그룹화 시도.

![Pasted image 20260221163645](../../GALLERY/Pasted%20image%2020260221163645.png)

그러나 위와 같이 v1 버전의 API 명세가 보이지 않음 

```java
@Bean  
public GroupedOpenApi v1Api() {  
    return GroupedOpenApi.builder()  
            .group("v1")  
            .pathsToMatch("/api/**", "/jwt/**")  
            .pathsToExclude("/api/v2/**", "/jwt/v2/**")  
            .build();  
}  
  
@Bean  
public GroupedOpenApi v2Api() {  
    return GroupedOpenApi.builder()  
            .group("v2")  
            .pathsToMatch("/api/v2/**", "/jwt/v2/**")  
            .build();  
}
```

원인은 바로 경로규칙이 다르기 때문이었다.  
추가로 `pathsToExclude` 를 통해 일부 API 경로들을 제한 할 수 있다.



# SecurityConfig 수정

```java
.requestMatchers("/v3/api-docs/**", "/swagger-ui/**", "/swagger-ui.html").permitAll()
```

springdocs, swagger는 시큐리티와 같이 자동적으로 경로가 생성되며 만약 Spring Security가 적용된 프로젝트라면 이에 대한 접근을 허용해야 한다.


# 어노테이션 학습

## 1) `@Tag`

컨트롤러(또는 API 묶음)에 **카테고리/섹션**을 붙입니다. Swagger UI에서 좌측 목록이 이 Tag 단위로 묶여 보여요.

```java
@Tag(name = "User", description = "유저 관련 API")
@RestController
@RequestMapping("/api/v1/users")
public class UserController { ... }
```

- `name`을 “도메인 단위(Users, Auth, Profile…)”로 통일하면 문서가 깔끔해집니다.

## 2) `@Operation`

**단일 엔드포인트(메서드)** 설명입니다. 제목/요약/상세/보안/요청·응답 설명을 여기에 적습니다.

```java
@Operation(
  summary = "내 정보 조회",
  description = "로그인된 사용자의 기본 정보를 조회합니다."
)
@GetMapping("/me")
public UserResponse me() { ... }
```

- `summary`는 짧게(한 줄), `description`은 부가 설명/제약/주의사항을 적는 용도로 쓰면 좋아요.

## 3) `@ApiResponses` / `@ApiResponse`

**응답 케이스(HTTP 상태코드별)** 문서화입니다. 성공/실패 응답을 나열합니다.

```java
@ApiResponses({
  @ApiResponse(responseCode = "200", description = "성공"),
  @ApiResponse(responseCode = "401", description = "인증 실패(토큰 없음/만료)"),
  @ApiResponse(responseCode = "403", description = "권한 없음"),
  @ApiResponse(responseCode = "404", description = "대상 없음")
})
```

- “실제 서비스에서 내려주는 에러 포맷(ErrorResponse DTO)”이 있다면, 아래 `@Content/@Schema`로 **실패 응답 body까지** 명확히 적는 게 핵심입니다.

## 4) `@Parameter`

**파라미터 설명**입니다. `@PathVariable`, `@RequestParam`, `@RequestHeader` 등에 붙여 “이 값이 뭔지/예시는 뭔지/필수인지”를 적습니다.

```java
@GetMapping("/{userId}")
public UserResponse get(
  @Parameter(description = "조회할 사용자 ID", example = "7", required = true)
  @PathVariable long userId
) { ... }
```

- `example`을 넣으면 Swagger UI에서 Try it out 할 때 엄청 편해집니다.

## 5) `@Content`

`@ApiResponse`(또는 `@RequestBody`) 안에서 **본문(body)의 미디어 타입과 스키마**를 정의합니다.  
즉, “이 응답은 JSON이고, JSON 구조는 어떤 DTO다”를 명시하는 곳입니다.

```java
@ApiResponse(
  responseCode = "200",
  description = "성공",
  content = @Content(
    mediaType = "application/json",
    schema = @Schema(implementation = UserResponse.class)
  )
)
```

- 성공/실패 응답이 서로 다른 DTO면 각각의 `@ApiResponse`에 다른 `@Content(schema=...)`를 붙입니다.

## 6) `@Schema`

DTO(클래스/필드)에 붙여 **모델(요청/응답 JSON)의 설명, 예시, 제약**을 적습니다. 문서 품질을 결정하는 핵심입니다.

```java
public record UserResponse(
  @Schema(description = "사용자 ID", example = "7")
  long id,

  @Schema(description = "로그인 아이디", example = "eojin")
  String loginId
) {}
```

- `@Schema`는 DTO 필드에 달아두면, 컨트롤러에서 매번 `@Content/@Schema(implementation=...)`를 안 적어도 기본 문서가 좋아집니다.

## 200 응답에 대한 자동추론

```java
@PostMapping("/api/routine")  
@Operation(summary = "루틴 생성")  
@ApiResponses(value = {  
        @ApiResponse(responseCode = "200", description = "생성 성공"),  
        @ApiResponse(  
                responseCode = "400",  
                description = "요청 검증 실패 (CommonErrorCode.INVALID_PARAMETER)",  
                content = @Content(schema = @Schema(implementation = ErrorResponse.class))  
        )  
})  
public ResponseEntity<RoutineResponse> createRoutine(@Valid @RequestBody RoutineRequest request) {  
    RoutineResponse response = routineService.createRoutine(request);  
    return ResponseEntity.ok(response);  
}
```

위와 같은 POST 요청에 대해 성공(200)에 대한 응답 DTO 형식이 명시되어 있지 않다.  
그러나 200에 `content`를 안적어놓으면 자동추론을 지원하여 정상적으로 잘 보인다.

![Pasted image 20260221170120](../../GALLERY/Pasted%20image%2020260221170120.png)

## 예외 응답 구체화

```java
@ApiResponse(  
        responseCode = "404",  
        description = "루틴 없음 (RoutineErrorCode.NOT_FOUND)",  
        content = @Content(schema = @Schema(implementation = ErrorResponse.class))  
)
```

```java
@Override  
@Transactional  
public RoutineDetailResponse findRoutine(long routineId) {  
  
    Routine routine = findRoutineById(routineId);  
    return routineMapper.toDetailDto(routine);  
}  
  
public Routine findRoutineById(long routineId) {  
    return routineRepository.findById(routineId)  
            .orElseThrow(() -> new RestApiException(RoutineErrorCode.NOT_FOUND, routineId));  
}
```

![Pasted image 20260221170825](../../GALLERY/Pasted%20image%2020260221170825.png)

현재 루틴 없음의 Example Value가 다음과 같이 표시되는데 좀 더 명확하게 명시할 수 있을까?

방법은 있었으나 커스텀 어노테이션을 만드는 등 다소 복잡함.  
또한 현재 단계에서 과하게 학습하다고 생각하여 적용하지 않음.  
다만 추후에 프로젝트에서 정말 사용한다면 해당 [링크](https://chatgpt.com/share/699969da-f7dc-8003-a436-cffabbe61a74)를 참고해보자.

