
## 🔍 1. 개념 요약

### ✅ **Bean Validation (JSR 380)**

- 자바 진영에서 **객체의 필드 값이 유효한지 검증하기 위한 표준 스펙**.
    
- `jakarta.validation` 패키지의 어노테이션을 사용.
    
- Spring Boot에서는 **`spring-boot-starter-validation`** 의존성을 통해 자동 통합됨.

### ✅ **주 목적**

- 클라이언트 요청(Request DTO)의 값이 **비어있거나, 잘못된 형식**일 때  
    Controller 로직에 들어오기 전에 **자동으로 예외를 발생시켜** 방어적 프로그래밍을 가능하게 함.

## 🧩 2. 동작 원리

1. **사용자가 API 요청**을 보냄 → 예: `POST /quiz-option`
    
2. **Spring Controller**에서 DTO를 `@RequestBody`로 매핑.
    
3. DTO에 붙은 `@NotBlank`, `@Size` 등의 **검증 어노테이션**을 기반으로  
    **`Validator`가 값의 유효성을 자동 체크.**
    
4. 검증에 실패하면 `MethodArgumentNotValidException`이 발생 →  
    `@ControllerAdvice`나 `@RestControllerAdvice`를 통해 예외 처리 가능.
    

---

## 🧱 3. 주요 어노테이션 정리

| 어노테이션             | 적용 대상    | 의미                  | 예시                               |
| ----------------- | -------- | ------------------- | -------------------------------- |
| `@Null`           | 모든 타입    | `null`만 허용함         | `@Null val nullable: null`       |
| `@NotNull`        | 모든 타입    | 값이 `null`이면 안 됨     | `@NotNull val age: Int`          |
| `@NotEmpty`       | 문자열, 컬렉션 | null ❌, empty("") ❌ | `@NotEmpty val name: String`     |
| `@NotBlank`       | 문자열      | null ❌, 공백만 ❌       | `@NotBlank val title: String`    |
| `@Size(min, max)` | 문자열, 컬렉션 | 길이 제한               | `@Size(max=50)`                  |
| `@Min`, `@Max`    | 숫자형      | 최소·최대값              | `@Min(0) @Max(100)`              |
| `@Email`          | 문자열      | 이메일 형식 검사           | `@Email val email: String`       |
| `@Pattern`        | 문자열      | 정규식 검사              | `@Pattern(regexp="^[a-zA-Z]+$")` |

---

## ⚙️ 4. Kotlin에서의 주의점 (`@field:` prefix)

Kotlin에서는 **프로퍼티 접근자(getter/setter)** 개념 때문에,  
어노테이션을 어디에 붙일지를 명시해야 합니다.

- `@field:NotBlank` → **필드**에 적용 (Spring Validation은 여기에 붙여야 작동)
    
- `@get:NotBlank` → **getter**에 적용
    
- `@set:NotBlank` → **setter**에 적용
    

✅ 즉, **Kotlin에서 Validation을 쓰려면 항상 `@field:`를 붙여야 정상 동작**합니다.

---

## 💡 5. 코틀린 : 실제 적용 예시

```kotlin
@RestController
@RequestMapping("/quiz-options")
class QuizOptionController {

    @PostMapping
    fun createOption(@Valid @RequestBody request: QuizOptionRequest): ResponseEntity<String> {
        // request.content, request.isCorrect는 이미 검증 완료됨
        return ResponseEntity.ok("선택지 등록 성공")
    }
}
```

- `@Valid` : Spring이 `QuizOptionRequest` 내부 어노테이션을 검사하도록 트리거함
    
- 검증 실패 시 → 자동으로 400 Bad Request + 에러 메시지 반환

---

## 🧠 6. 핵심 요약

| 항목         | 설명                                                       |
| ---------- | -------------------------------------------------------- |
| 목적         | DTO의 입력값 유효성 검사                                          |
| 표준         | Jakarta Bean Validation (JSR 380)                        |
| 주요 어노테이션   | `@NotNull`, `@NotBlank`, `@Size`, `@Email`, `@Pattern` 등 |
| Kotlin 주의점 | `@field:` prefix 필요                                      |
| 동작 위치      | Controller 진입 전 자동 검증                                    |
| 결과         | 유효성 실패 시 예외 발생 (`MethodArgumentNotValidException`)       |


## 추가 어노테이션
![Pasted image 20251111214521](../../GALLERY/Pasted%20image%2020251111214521.png)
[출처-티스토리](https://dev-coco.tistory.com/123)

