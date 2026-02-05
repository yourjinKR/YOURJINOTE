
## Spring Security 정책 문서 (현재 적용된 상태)

### 1) 목표/전제

- **Stateless 인증**을 전제로 하며, 서버 세션을 생성하지 않고 JWT로 인증을 유지합니다.
    
- 기본 보안 메커니즘 중
    
    - CSRF 비활성화
        
    - FormLogin 비활성화
        
    - HTTP Basic 비활성화  
        형태로 구성되어 있습니다.
        

---

### 2) CORS 정책

- 허용 Origin: `http://localhost:5173`
    
- 허용 Method: `GET, POST, PUT, DELETE, OPTIONS`
    
- 허용 Header: `Authorization`, `Content-Type`, `Accept`
    
- 노출 Header: `Authorization`, `Set-Cookie`
    
- **AllowCredentials: false** (현재 설정)
    

---

### 3) 인가(Authorization) 정책

다음 경로는 인증 없이 접근 허용(permitAll)입니다.

- `/jwt/exchange`, `/jwt/refresh`
    
- `POST /api/user/exist`, `POST /api/user`
    
- `GET /api/check`
    

다음 경로는 **Role 기반 접근 제어**입니다.

- `GET /api/user/me` : `MEMBER` 또는 `TRAINER` 또는 `ADMIN` 역할 필요
    

그 외 모든 요청:

- **인증 필요(authenticated)**
    

---

### 4) 권한(Role) 계층 정책

- **ADMIN > (MEMBER, TRAINER)** 역할 계층을 사용합니다.
    

---

### 5) 예외 처리 정책

- 인증 실패(401): `AuthenticationEntryPoint`에서 401 응답
    
- 인가 실패(403): `AccessDeniedHandler`에서 403 응답
    

---

### 6) 필터 체인 구성 (요청 처리 순서)

SecurityFilterChain에서 커스텀 필터가 다음 순서로 추가되어 있습니다.

1. **JwtFilter**
    
    - `LogoutFilter` 이전에 실행되도록 배치
        
2. **LoginFilter**
    
    - `UsernamePasswordAuthenticationFilter` 이전에 실행되도록 배치
        

---

## 인증(Authentication) 플로우 상세

### A) JWT 인증 (일반 API 요청)

- 요청 헤더에서 `Authorization`을 읽고, `Bearer` 로 시작하면 토큰을 파싱합니다.
    
- AccessToken이 유효하면(username, role 추출) `SecurityContext`에 Authentication을 세팅합니다.
    
- 유효하지 않으면 401 + JSON 에러 응답을 직접 반환합니다.
    

---

### B) 자체 로그인 (POST /login)

- `LoginFilter`가 `POST /login`을 가로채고, JSON body에서 `loginId/password`를 읽어 인증을 시도합니다.
    
- 인증 성공 시 `LoginSuccessHandler`가:
    
    - Access/Refresh 토큰 모두 발급
        
    - Refresh 토큰을 DB(whitelist) 저장
        
    - 응답으로 `{accessToken, refreshToken}` JSON 반환
        

---

### C) 소셜 로그인 (OAuth2)

- `oauth2Login()` 성공 시 `SocialSuccessHandler`로 이동합니다.
    
- `SocialSuccessHandler`가:
    
    - **Refresh 토큰만 발급**
        
    - Refresh 토큰을 DB whitelist 저장
        
    - `refreshToken`을 **HttpOnly 쿠키**로 내려줌
        
    - 이후 `http://localhost:5173/cookie`로 redirect
        

---

### D) 로그아웃

- 로그아웃 처리 시 `RefreshTokenLogoutHandler`가 추가되어 있습니다.
    
- 이 핸들러는 요청 body의 `refreshToken`을 읽고, 유효하면 DB whitelist에서 삭제합니다.
    

---

# 3가지 이슈와 고치는 방법

아래 3개는 **현재 코드 그대로 두면 실제로 장애/버그로 이어질 확률이 높은 포인트**입니다.

---

## 이슈 1) 소셜 로그인 role prefix가 꼬일 가능성 (ROLE_ROLE_… 문제)

### 왜 문제인가?

- 소셜 성공 핸들러는 role을 `authentication.getAuthorities().iterator().next().getAuthority()`로 받은 뒤, 토큰 생성 시 `"ROLE_" + role`을 다시 붙입니다.
    
- 그런데 `getAuthority()` 값이 이미 `ROLE_MEMBER` 같은 형태라면 최종적으로 `ROLE_ROLE_MEMBER`가 됩니다.
    
- 반면 자체 로그인 성공 핸들러는 role을 그대로 토큰에 넣고 있습니다.
    

➡️ 결과적으로 **자체 로그인 토큰과 소셜 로그인 토큰의 role 포맷이 달라져서**, `hasAnyRole("MEMBER", ...)` 같은 인가가 한쪽만 실패할 수 있습니다.

### 해결 방법(추천: “권한 문자열을 한 규칙으로 통일”)

**정답은 하나입니다: 토큰에 들어가는 role은 항상 `ROLE_XXX` 형태로 통일하세요.**

- `SocialSuccessHandler` 수정
    
    - `JwtUtil.createJWT(username, role, false);` 로 변경 (ROLE_를 붙이지 않기)
        
- `LoginSuccessHandler`도 동일 규칙 유지(지금도 role을 그대로 넣고 있음)
    

추가로 더 안전한 방식:

- role이 `ROLE_`를 포함하는지 체크해서 중복 prefix를 방지하는 “정규화 함수”를 하나 두세요.
    
    - 예: `normalizeRole(role) -> role.startsWith("ROLE_") ? role : "ROLE_" + role`
        

---

## 이슈 2) CORS allowCredentials=false인데 소셜은 쿠키 기반이라 크로스 오리진에서 쿠키가 안 실릴 수 있음

### 왜 문제인가?

- CORS 설정이 `allowCredentials(false)`입니다.
    
- 하지만 소셜 로그인은 refreshToken을 쿠키로 내려주고 있습니다.
    

➡️ 프론트(5173) ↔ 백엔드(다른 포트/도메인) 환경에서 “쿠키 기반 인증”을 하려면 보통 **credentials 허용**이 필요합니다. 지금 설정은 쿠키 전송 시나리오와 충돌하기 쉬워요.

### 해결 방법(쿠키 기반을 유지한다면)

1. 서버 CORS 설정 변경
    

- `configuration.setAllowCredentials(true);` 로 변경
    
- allowedOrigins는 이미 구체적으로 지정되어 있어 OK (`http://localhost:5173`)
    

2. 프론트 요청도 credentials 포함
    

- fetch: `credentials: "include"`
    
- axios: `withCredentials: true`
    

3. (운영환경 필수) SameSite / Secure 정리
    

- 지금 쿠키는 `setSecure(false)`입니다.
    
- 운영에서 HTTPS를 쓸 거라면:
    
    - `Secure=true`, `SameSite=None` 조합이 필요해지는 경우가 많습니다(크로스 사이트 쿠키 전송 시).
        
    - 단, 서블릿 `Cookie`만으로 SameSite를 제어하기 어렵기 때문에 `Set-Cookie` 헤더를 직접 구성하거나 ResponseCookie(스프링) 사용을 고려하세요.
        

> 만약 “쿠키 기반을 포기하고 헤더(Bearer)만 쓰겠다”라면: 소셜 성공 시에도 **accessToken을 JSON으로 내려주거나**, redirect 후 프론트가 토큰을 받아 헤더로 전환하는 구조로 통일하는 게 더 단순해집니다.

---

## 이슈 3) refreshToken 쿠키 maxAge=10초라서 플로우 중간에 쉽게 만료됨

### 왜 문제인가?

- 소셜 성공 핸들러에서 refreshToken 쿠키의 maxAge가 **10초**입니다.
    
- redirect → 프론트 페이지 로딩 → exchange 호출 과정에서 조금만 지연돼도 쿠키가 만료될 수 있습니다.
    

### 해결 방법

- refreshToken 만료 시간(예: 7일/14일 등)과 쿠키 maxAge를 **동일한 정책으로 맞추세요.**
    
- 하드코딩 대신 설정값으로 빼는 것을 추천합니다.
    
    - 예: `security.jwt.refresh-cookie-max-age-seconds`
        

추가로 같이 고치면 좋은 점(운영 안정성)

- 자체 로그인은 refresh를 JSON으로 주고, 소셜은 쿠키로 줍니다.
    
- 둘 중 하나로 통일하면 디버깅/클라이언트 구현 난도가 크게 내려갑니다.
    
    - **추천 1:** 둘 다 HttpOnly 쿠키(Access는 헤더, Refresh는 쿠키)
        
    - **추천 2:** 둘 다 JSON 응답(쿠키 사용 최소화)
        

---

# 보너스: “바로 개선하면 좋은” 2개(선택)

1. **Logout 시 refreshToken 입력 경로 통일**
    

- 지금은 body에서만 읽습니다.
    
- 소셜은 쿠키에 refresh가 있으니, 쿠키에서도 읽어오게 하면 UX/일관성이 좋아집니다.
    

2. **JwtFilter의 Bearer 처리**
    

- Bearer가 아니면 `ServletException`을 던지고 있는데, 현재 예외 처리 흐름에서 어떻게 응답될지(500으로 번질지) 점검이 필요합니다.
    
- “일관된 401 JSON”을 유지하려면 Bearer가 아니어도 401로 정리하는 편이 깔끔합니다.
    

---

원하시면, 위 문서를 그대로 **프로젝트 문서/포트폴리오/면접용 설명**으로 쓸 수 있게:

- “1분 요약(면접 답변)”
    
- “왜 Stateless + JWT + Refresh whitelist를 택했는지”
    
- “소셜 로그인 redirect/exchange 설계 의도”
    

까지 한 번에 정리해서 드리겠습니다.