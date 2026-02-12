# TanStack Query

리엑트 쿼리는 서버에서 가져온 데이터를 웹 브라우저 앱에서 사용하기 쉽게 도와주는 기술이다.

데이터베이스에서 가져온 데이터를 클라이언트에서 보여주기 위해 우리는 [ajax](../JAVASCRIPT/AJAX.md)를 이용한다, 이 때 서버에서 가져오는 데이터를 `서버의 상태` 라고 한다.

TanStack Query는 서버로부터 데이터 가져오기, 데이터 캐싱, 캐시 제어 등 데이터를 쉽고 효율적으로 관리할 수 있는 라이브러리이다.

### 지원 기능

- 데이터 가져오기 및 캐싱
- 동일 요청의 중복 제거
- 신선한 데이터 유지
- 무한 스크롤, 페이지네이션 등의 성능 최적화
- 네트워크 재연결, 요청 실패 등의 자동 갱신

# 데이터 캐싱

데이터를 가져올때는 항상 쿼리 키(`queryKey`)를 지정해야 한다.

```ts
import { useQuery } from '@tanstack/react-query'

export default function DelayedData() {
  const { data } = useQuery({
    queryKey: ['delay'], // <- 쿼리 키
    queryFn: async () => (await fetch('https://api.heropy.dev/v0/delay?t=1000')).json()
  })
  return <div>{JSON.stringify(data)}</div>
}
```

## queryKey

캐시된 데이터와 비교해 새로운 데이터를 가져올지, 캐시된 데이터를 사용할지 결정하는 기준이 된다.

아래와 같이 feature별로 쿼리 키를 통합적으로 관리할 수 있다.

```ts
export const profileQueryKeys = {
  all: ["profile"] as const,
  myProfile: () => [...profileQueryKeys.all, "me"] as const,
  inbody: (request: InBodyRecordRequest) => [...profileQueryKeys.all, "inbody", request] as const,
} as const;
```

## queryFn

쿼리 함수(`queryFn`)는 데이터를 가져오는 비동기 함수로, **꼭 데이터를 반환하거나 오류를 던져야 한다.** 
던져진 오류는 반환되는 `error` 객체로 확인할 수 있습니다.  

> `error`는 기본적으로 `null`이다.

```ts
queryFn: () => getUserMyProfile()
```

```ts
export const getUserMyProfile = async (): Promise<UserWithProfileResponse> => {
  const response = await api.get<UserWithProfileResponse>("/api/user/profile/me");
  return response.data;
};
```

# useQuery

가장 기본적인 쿼리 훅으로, 컴포넌트에서 데이터를 가져올 때 사용합니다.

```tsx
const 반환 = useQuery<데이터타입>(옵션)
```

# useMutation

TanStack Query는 데이터 변경 작업(생성, 수정, 삭제 등)을 위한 `useMutation` 훅을 제공

> 쿼리(`useQuery`)는 '가져오기'에 집중하는 반면, 변이(`useMutation`)는 '보내기'에 집중하는 훅으로 이해하면 쉽습니다.

```tsx
const 반환 = useMutation(옵션)
```

### onSuccess

`onSuccess`는 3가지 인자를 순서대로 받습니다.

1. `data`: 서버로부터 반환된 **응답 데이터** (Response Body).
    
2. `variables`: `mutate()` 함수를 호출할 때 넘겨준 **요청 변수** (Payload).
    
3. `context`: `onMutate` 콜백에서 반환한 값 (주로 롤백용 데이터).

```tsx
/**
 * 프로필 생성 mutation hook
 * 프로필을 생성하고 성공 시 프로필 쿼리 캐시를 무효화
 *
 * @returns {Object} 프로필 생성 mutation 객체
 *   - mutate: 프로필 생성 실행 함수
 *   - isPending: 생성 중 여부
 *   - isError: 에러 발생 여부
 *   - error: 에러 정보
 */
export const useCreateProfileMutation = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (request: UserProfileRequest) => createUserProfile(request),
    onSuccess: () => {
      // 프로필 생성 후 현재 사용자의 프로필 쿼리 캐시 무효화
      // 다음 useMyProfileQuery 호출 시 서버에서 최신 데이터 조회
      queryClient.invalidateQueries({
        queryKey: profileQueryKeys.myProfile(),
      });
    },
  });
};

```

<br>

# 출처

[테코톡](https://www.youtube.com/watch?v=RfK15tw8H-I)  
[velog - 리액트 쿼리](https://velog.io/@jay/10-minute-react-query-concept)  
[heropy.dev](https://www.heropy.dev/p/HZaKIE)  

