# PUT과 PATCH의 차이

[HTTP 메서드](HTTP%20METHOD.md)에서 PUT과 PATCH는 둘 다 리소스의 상태를 수정할때 사용된다.    
그러나 둘의 차이점의 핵심은 [멱등성](멱등성.md)의 차이이다.  

## 정리

### PUT과 PATCH의 차이점

PUT은 리소스를 요청 바디로 **완전히 대체**하는 메서드이고,  
PATCH는 리소스의 **일부 속성만 수정**할 때 사용합니다.  
실무에서는 대부분 부분 수정이 많기 때문에 PATCH를 더 자주 사용하며,  
전체 상태를 항상 전달하는 관리자성 수정이나 리소스 동기화 목적일 때 PUT을 사용합니다.


> PUT 메서드와 PATCH 메서드의 주요 차이점은 PUT 메서드는 요청 URI를 사용하여 요청된 리소스의 수정된 버전을 제공하여 리소스의 원래 버전을 대체하는 반면, PATCH 메서드는 리소스를 수정하기 위한 일련의 지침을 제공한다는 것입니다. PATCH 문서가 PUT 메서드 로 전송된 리소스의 새 버전 크기보다 큰 경우 PUT 메서드가 선호됩니다.  
> - 위키피디아-

### PATCH와 멱등성

PATCH는 부분 수정이기 때문에 멱등하지 않다고 오해되지만,  실제로는 **연산 기반으로 설계할 경우에만 비멱등**합니다.  값을 명시적으로 설정하는 방식으로 설계하면  PATCH도 충분히 멱등성을 보장할 수 있고,  
실무에서는 이 방식이 일반적입니다.


## PUT 

> 해당 리소스를 요청 바디로 완전히 대체

PUT은 수정 요청을 덮어쓰기 방식으로 진행한다.

- 멱등성 보장
- 클라이언트가 리소스의 전체 상태를 알고 있을 때 적합
- 관리자 페이지 에서 전체 수정 폼, 엔티티 전체를 내려보낼때, DTO <-> Entity 완벽 매칭시

> RFC 5789 명세에 따르면 PUT은 반드시 멱등해야 한다.

## PATCH

> 리소스의 일부분만 변경

PATCH는 리소스를 요청에 포함된 필드만 부분 수정합니다.

- 일반적으로 멱등성을 보장하지 않을 수 있음
- 부분 업데이트에 최적화
- 네트워크 비용 감소
- 프로필  수정, 상태 변경, 모달/인라인 수정 UI, 모바일 환경

> PATCH == 멱등하지 않음 👈 해당 개념은 옳지 않음!!!
### PATCH로 멱등성을 보장하는 법

얘를 들어 `포인트를 100 추가해라` 와 같은 요청은 전혀 멱등하지 않다.  
다른 예시로는 `상태를 토글해라` 와 같은 요청 또한 멱등하지 않다.

그러나 `상태를 A로 만들어라`와 같은 요청은 멱등하다. 그리고 이와 같은 요청은 PATCH에 해당할 것이다.

> PATCH를 “변경하라”가 아니라 “이 상태가 되게 하라”로 설계하면 멱등하다

#### JPA 예시

```java
@Getter
public class UserPatchRequest {
    private String name;
    private Integer age;
    private Boolean enabled;
}
```

```java
@Transactional
public void patchUser(Long id, UserPatchRequest dto) {
    User user = userRepository.findById(id)
        .orElseThrow();

    if (dto.getName() != null) {
        user.setName(dto.getName());
    }
    if (dto.getAge() != null) {
        user.setAge(dto.getAge());
    }
    if (dto.getEnabled() != null) {
        user.setEnabled(dto.getEnabled());
    }
}

```