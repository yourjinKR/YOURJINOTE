
## 트러블 슈팅 일지: 소셜 로그인 후 `/api/user/me` 401/403 발생

> 소셜 로그인 후 `/api/user/me` 호출이 실패하던 이슈를 Spring Security DEBUG 로그로 추적해 **ROLE prefix 제거 상태에서 `hasAnyRole()`을 사용해 발생한 권한 불일치(403)** 를 확인하고, **`hasAuthority()`로 인가 정책을 통일 + `/error` 허용으로 원인 노출**까지 정리해 정상 동작으로 복구했습니다.


### 1) 현상

- 소셜 로그인(카카오) 성공 이후, 내 정보 조회 API `GET /api/user/me` 호출 시 응답이 실패함.
    
- 초기에는 **401 Unauthorized**로 보였고, `/error` 경로를 허용한 후에는 **403 Forbidden**으로 확인됨.
    
- 프론트 요청 헤더에는 다음 인증 정보가 포함되어 있었음:
    
    - `Authorization: Bearer <access_token>`
        
    - `Cookie: JSESSIONID=<...>`
        

### 2) 원인 분석 과정

- Spring Security DEBUG 로그를 통해 OAuth2 로그인 자체는 정상적으로 인증 컨텍스트가 생성되는 것을 확인:
    
    - `OAuth2AuthenticationToken`, `Granted Authorities=[MEMBER]`
        
- 그러나 `/api/user/me` 요청에서 컨트롤러 매핑 로그가 찍히지 않고 `/error`로 디스패치되는 흐름을 확인.
    
- `/error`를 허용하기 전에는 **실제 원인(403)이 `/error` 보안 처리로 가려져** 401처럼 보이며 디버깅이 어려웠음.
    

### 3) 핵심 원인

- 프로젝트에서 **ROLE prefix(`ROLE_`)를 제거한 상태**로 권한을 운용하고 있었음.
    
    - 예: 권한이 `MEMBER` 형태로 저장/부여됨
        
- 반면 인가 설정에서는 `hasAnyRole("MEMBER")` 같은 **Role 기반 검사**를 사용하고 있었음.
    
    - `hasRole/hasAnyRole`은 내부적으로 `ROLE_MEMBER`를 요구함
        
    - 결과적으로 실제 권한(`MEMBER`)과 요구 권한(`ROLE_MEMBER`)이 불일치하여 **403 Forbidden** 발생
        

### 4) 해결

- 권한 검사 방식과 권한 문자열 정책을 일치시킴.
    
    - (선택지 예시)
        
        - 권한이 `MEMBER`라면 → `hasAuthority("MEMBER")` / `hasAnyAuthority(...)` 사용
            
        - `hasAnyRole(...)`를 쓰고 싶다면 → 권한을 `ROLE_MEMBER` 형태로 부여(ROLE prefix 유지)
            
- 디버깅 편의성을 위해 `/error` 엔드포인트를 permitAll 처리하여,
    
    - 에러가 다시 보안 처리로 가려지지 않도록 조치함.
        

### 5) 배운 점 / 재발 방지 체크리스트

- **권한 문자열 정책(ROLE prefix 유무)**과 **인가 API(`hasRole` vs `hasAuthority`)**는 반드시 한 세트로 맞춰야 한다.
    
- 인증/인가 실패처럼 보이는 현상도 실제로는 **에러 디스패치(`/error`)가 보안에 걸려 상태 코드가 왜곡**될 수 있다.
    
- Spring Security DEBUG 로그는 “필터 체인에서 어느 단계에서 실패하는지” 확인하는 데 매우 유용하다.
    
- 신규 엔드포인트 추가 시:
    
    - `permitAll` 대상(`/error`, health check 등)과
        
    - 보호 대상(`/api/**`)의 권한 조건을 **일관된 규칙으로** 검토한다.
        
