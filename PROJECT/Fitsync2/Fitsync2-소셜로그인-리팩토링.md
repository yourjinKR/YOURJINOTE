
## 1) “정규화된 소셜 사용자 정보” 모델

```java
package app.fitsync.domain.user.oauth;

import app.fitsync.domain.user.entity.SocialProviderType;
import java.util.Map;

public record SocialUserInfo(
        SocialProviderType provider,
        String providerUserId,      // 네이버 id / 구글 sub / 카카오 id
        String email,               // optional
        String name,                // optional
        Map<String, Object> rawAttributes
) {
    public String loginId() {
        // 우리 서비스 식별자 규칙은 공통으로!
        return provider.name() + "_" + providerUserId;
    }
}
```


## 2) Provider별 “추출기(Extractor)” 인터페이스

```java
package app.fitsync.domain.user.oauth;

import app.fitsync.domain.user.entity.SocialProviderType;
import org.springframework.security.oauth2.core.user.OAuth2User;

public interface SocialUserInfoExtractor {
    SocialProviderType supports();

    SocialUserInfo extract(OAuth2User oAuth2User);
}
```


## 3) 공통 유틸 (널/캐스팅 안전 처리)

```java
package app.fitsync.domain.user.oauth;

import java.util.Map;

public final class AttrUtils {
    private AttrUtils() {}

    @SuppressWarnings("unchecked")
    public static Map<String, Object> asMap(Object value, String keyNameForError) {
        if (value == null) {
            throw new IllegalArgumentException("Missing map attribute: " + keyNameForError);
        }
        if (!(value instanceof Map)) {
            throw new IllegalArgumentException("Attribute is not a map: " + keyNameForError);
        }
        return (Map<String, Object>) value;
    }

    public static String str(Object value) {
        return value == null ? null : String.valueOf(value);
    }

    public static String requiredStr(Object value, String keyNameForError) {
        String s = str(value);
        if (s == null || s.isBlank()) {
            throw new IllegalArgumentException("Missing required attribute: " + keyNameForError);
        }
        return s;
    }
}
```


## 4) Provider별 Extractor 구현체들

### 4-1) NAVER

```java
package app.fitsync.domain.user.oauth.extractor;

import app.fitsync.domain.user.entity.SocialProviderType;
import app.fitsync.domain.user.oauth.AttrUtils;
import app.fitsync.domain.user.oauth.SocialUserInfo;
import app.fitsync.domain.user.oauth.SocialUserInfoExtractor;
import java.util.Map;
import org.springframework.stereotype.Component;
import org.springframework.security.oauth2.core.user.OAuth2User;

@Component
public class NaverUserInfoExtractor implements SocialUserInfoExtractor {

    @Override
    public SocialProviderType supports() {
        return SocialProviderType.NAVER;
    }

    @Override
    public SocialUserInfo extract(OAuth2User oAuth2User) {
        Map<String, Object> root = oAuth2User.getAttributes();
        Map<String, Object> response = AttrUtils.asMap(root.get("response"), "response");

        String providerUserId = AttrUtils.requiredStr(response.get("id"), "response.id");
        String email = AttrUtils.str(response.get("email"));
        String name  = AttrUtils.str(response.get("name"));

        // principal에 넣을 raw를 "정규화된 형태"로 넣고 싶다면 여기서 재구성해도 됩니다.
        return new SocialUserInfo(SocialProviderType.NAVER, providerUserId, email, name, response);
    }
}
```

### 4-2) GOOGLE

```java
package app.fitsync.domain.user.oauth.extractor;

import app.fitsync.domain.user.entity.SocialProviderType;
import app.fitsync.domain.user.oauth.AttrUtils;
import app.fitsync.domain.user.oauth.SocialUserInfo;
import app.fitsync.domain.user.oauth.SocialUserInfoExtractor;
import java.util.Map;
import org.springframework.stereotype.Component;
import org.springframework.security.oauth2.core.user.OAuth2User;

@Component
public class GoogleUserInfoExtractor implements SocialUserInfoExtractor {

    @Override
    public SocialProviderType supports() {
        return SocialProviderType.GOOGLE;
    }

    @Override
    public SocialUserInfo extract(OAuth2User oAuth2User) {
        Map<String, Object> attr = oAuth2User.getAttributes();

        String providerUserId = AttrUtils.requiredStr(attr.get("sub"), "sub");
        String email = AttrUtils.str(attr.get("email"));
        String name  = AttrUtils.str(attr.get("name"));

        return new SocialUserInfo(SocialProviderType.GOOGLE, providerUserId, email, name, attr);
    }
}
```

### 4-3) KAKAO

```java
package app.fitsync.domain.user.oauth.extractor;

import app.fitsync.domain.user.entity.SocialProviderType;
import app.fitsync.domain.user.oauth.AttrUtils;
import app.fitsync.domain.user.oauth.SocialUserInfo;
import app.fitsync.domain.user.oauth.SocialUserInfoExtractor;
import java.util.Map;
import org.springframework.stereotype.Component;
import org.springframework.security.oauth2.core.user.OAuth2User;

@Component
public class KakaoUserInfoExtractor implements SocialUserInfoExtractor {

    @Override
    public SocialProviderType supports() {
        return SocialProviderType.KAKAO;
    }

    @Override
    public SocialUserInfo extract(OAuth2User oAuth2User) {
        Map<String, Object> attr = oAuth2User.getAttributes();

        // 카카오 id는 Long일 수 있어 String.valueOf로 안전 처리
        String providerUserId = AttrUtils.requiredStr(attr.get("id"), "id");

        Map<String, Object> kakaoAccount = AttrUtils.asMap(attr.get("kakao_account"), "kakao_account");

        // 카카오는 profile 안에 nickname이 있는 경우가 흔함
        String email = AttrUtils.str(kakaoAccount.get("email"));

        String name = AttrUtils.str(kakaoAccount.get("name"));
        if (name == null) {
            Object profileObj = kakaoAccount.get("profile");
            if (profileObj instanceof Map<?, ?> profile) {
                name = AttrUtils.str(profile.get("nickname"));
            }
        }

        // raw는 kakao_account만 넣을 수도 있고, 전체 attr을 넣을 수도 있습니다.
        // 디버깅/추적을 위해 전체 attr을 넣는 것도 좋습니다.
        return new SocialUserInfo(SocialProviderType.KAKAO, providerUserId, email, name, attr);
    }
}
```

## 5) Extractor Registry (provider → extractor 매핑)

```java
package app.fitsync.domain.user.oauth;

import app.fitsync.domain.user.entity.SocialProviderType;
import java.util.EnumMap;
import java.util.List;
import java.util.Map;
import org.springframework.stereotype.Component;

@Component
public class SocialUserInfoExtractorRegistry {

    private final Map<SocialProviderType, SocialUserInfoExtractor> map;

    public SocialUserInfoExtractorRegistry(List<SocialUserInfoExtractor> extractors) {
        EnumMap<SocialProviderType, SocialUserInfoExtractor> tmp = new EnumMap<>(SocialProviderType.class);
        for (SocialUserInfoExtractor ex : extractors) {
            SocialProviderType type = ex.supports();
            if (tmp.containsKey(type)) {
                throw new IllegalStateException("Duplicate extractor for provider: " + type);
            }
            tmp.put(type, ex);
        }
        this.map = tmp;
    }

    public SocialUserInfoExtractor get(SocialProviderType type) {
        SocialUserInfoExtractor ex = map.get(type);
        if (ex == null) {
            throw new IllegalArgumentException("Unsupported social provider: " + type);
        }
        return ex;
    }
}
```

## 6) Provider 문자열 → Enum 변환기 (한 군데에서만 처리)

```java
package app.fitsync.domain.user.oauth;

import app.fitsync.domain.user.entity.SocialProviderType;

public final class SocialProviderResolver {
    private SocialProviderResolver() {}

    public static SocialProviderType fromRegistrationId(String registrationId) {
        if (registrationId == null || registrationId.isBlank()) {
            throw new IllegalArgumentException("registrationId is empty");
        }
        return SocialProviderType.valueOf(registrationId.trim().toUpperCase());
    }
}
```

## 7) Principal (정규화된 attributes + loginId + authorities)

```java
package app.fitsync.domain.user.dto;

import java.util.Collection;
import java.util.Map;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.oauth2.core.user.OAuth2User;

public class CustomOAuth2UserV2 implements OAuth2User {

    private final Map<String, Object> attributes;
    private final Collection<? extends GrantedAuthority> authorities;
    private final String loginId;

    public CustomOAuth2UserV2(Map<String, Object> attributes,
                              Collection<? extends GrantedAuthority> authorities,
                              String loginId) {
        this.attributes = attributes;
        this.authorities = authorities;
        this.loginId = loginId;
    }

    @Override
    public Map<String, Object> getAttributes() {
        return attributes;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    @Override
    public String getName() {
        // Spring Security에서 “principal name” 역할
        return loginId;
    }
}
```

## 8) 최종: CustomOAuth2UserService (loadUser 리팩토링 완료본)

```java
package app.fitsync.domain.user.service;

import app.fitsync.domain.user.dto.CustomOAuth2UserV2;
import app.fitsync.domain.user.entity.SocialProviderType;
import app.fitsync.domain.user.entity.User;
import app.fitsync.domain.user.entity.UserRoleType;
import app.fitsync.domain.user.oauth.SocialProviderResolver;
import app.fitsync.domain.user.oauth.SocialUserInfo;
import app.fitsync.domain.user.oauth.SocialUserInfoExtractor;
import app.fitsync.domain.user.oauth.SocialUserInfoExtractorRegistry;
import app.fitsync.domain.user.repository.UserRepository;
import java.util.List;
import java.util.Optional;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.client.userinfo.DefaultOAuth2UserService;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserRequest;
import org.springframework.security.oauth2.core.OAuth2AuthenticationException;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.stereotype.Service;

@Service
public class CustomOAuth2UserServiceV2 extends DefaultOAuth2UserService {

    private final UserRepository userRepository;
    private final SocialUserInfoExtractorRegistry extractorRegistry;

    public CustomOAuth2UserServiceV2(UserRepository userRepository,
                                    SocialUserInfoExtractorRegistry extractorRegistry) {
        this.userRepository = userRepository;
        this.extractorRegistry = extractorRegistry;
    }

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2User oAuth2User = super.loadUser(userRequest);

        // 1) provider 결정
        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        SocialProviderType providerType = SocialProviderResolver.fromRegistrationId(registrationId);

        // 2) provider별 추출(정규화)
        SocialUserInfoExtractor extractor = extractorRegistry.get(providerType);
        SocialUserInfo info = extractor.extract(oAuth2User);

        // 3) 공통 upsert
        User user = upsertSocialUser(info);

        // 4) 권한 & principal
        String role = user.getRoleType() != null ? user.getRoleType().name() : UserRoleType.MEMBER.name();
        List<GrantedAuthority> authorities = List.of(new SimpleGrantedAuthority(role));

        // principal에 넣을 attributes는 "정규화된 것"을 권장
        // 지금은 extractor가 내려준 rawAttributes를 그대로 넣되, 필요하면 표준 Map으로 바꾸세요.
        return new CustomOAuth2UserV2(info.rawAttributes(), authorities, user.getLoginId());
    }

    private User upsertSocialUser(SocialUserInfo info) {
        String loginId = info.loginId();

        Optional<User> found = userRepository.findByLoginIdAndIsSocial(loginId, true);

        if (found.isPresent()) {
            User user = found.get();

            // ✅ 업데이트 정책: null/blank로 덮어쓰지 않기
            // (아래 필드명/메서드는 프로젝트에 맞게 조정)
            boolean changed = false;

            if (info.email() != null && !info.email().isBlank()) {
                // user.setEmail(info.email());
                changed = true;
            }
            if (info.name() != null && !info.name().isBlank()) {
                // user.setName(info.name());
                changed = true;
            }

            return changed ? userRepository.save(user) : user;
        }

        // 신규 가입
        User newUser = User.builder()
                .loginId(loginId)
                .password("") // 소셜 유저는 비밀번호 미사용 정책이면 "" 또는 null 정책 결정
                .name(info.name() != null ? info.name() : "Unknown") // 정책: 없으면 임시값
                .roleType(UserRoleType.MEMBER)
                .email(info.email())
                .isSocial(true)
                .socialProviderType(info.provider())
                .build();

        return userRepository.save(newUser);
    }
}
```

# 이 구조의 장점 (당장 체감되는 포인트)

- `loadUser()`에서 **if-else 완전 제거**
    
- provider 추가 시: **Extractor 1개 추가하면 끝**
    
- loginId 규칙 / upsert 정책이 **한 곳**에 모여서 일관성 유지
    
- “email/name이 없을 수 있음” 같은 운영 이슈를 **정책**으로 관리 가능
    

원하시면 다음 단계로 더 깔끔하게 만들어드릴 수도 있어요:

1. principal attributes를 아예 `Map.of("provider",..., "loginId",..., "email",...)`처럼 **표준 Map**으로 고정
    
2. upsert 정책을 `SocialUserUpdater` 같은 별도 클래스로 분리해서 테스트 용이하게 만들기
    
3. “추가 정보 입력이 필요한 경우(이메일 없음 등)”를 예외로 분기해서 프론트로 보내는 흐름 정리