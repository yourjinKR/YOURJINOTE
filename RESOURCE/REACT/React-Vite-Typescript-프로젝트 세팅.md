
# 프로젝트 생성 및 기본 실행

```shell
npm create vite@latest my-app -- --template react-ts
cd my-app
npm i
npm run dev
```

# 폴더 구조



# GPT

좋습니다. 지금 `package.json`이면 “styled-components 기반 + Spring 연동 + 개발 유틸”까지 **바로 뼈대 잡는 세팅**을 아래대로 넣으시면 됩니다. (코드/파일 그대로 복붙 가능)

---

## 0) 패키지 2개만 추가 설치 (필수)

styled-components를 TS에서 제대로 쓰려면 타입이 필요하고, 전역 스타일/테마도 자연스럽게 잡습니다.

```bash
npm i -D @types/styled-components
```

(선택) axios 요청/응답 타입을 깔끔히 하려면 zod도 좋지만, 지금은 생략해도 됩니다.

---

## 1) Vite 프록시로 Spring `/api` 붙이기

`vite.config.ts`

```ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      "/api": {
        target: "http://localhost:8080",
        changeOrigin: true
      }
    }
  }
});
```

프론트에선 항상 `/api/...` 로 호출하면 dev 환경에서 CORS 없이 동작합니다.

---

## 2) 환경변수 (prod 대비)

`.env.development`

```
VITE_API_BASE_URL=/api
```

`.env.production`

```
VITE_API_BASE_URL=https://your-domain.com/api
```

---

## 3) axios 인스턴스 + 공통 에러 처리 골격

`src/shared/api/http.ts`

```ts
import axios from "axios";

export const http = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 15_000,
  withCredentials: true // 세션/쿠키 기반이면 true 유지, JWT만 쓰면 false로
});

// 필요 시 토큰/에러 공통 처리
http.interceptors.response.use(
  (res) => res,
  (err) => {
    // TODO: 401 처리(로그아웃/리프레시), 500 토스트 등 공통 처리 위치
    return Promise.reject(err);
  }
);
```

---

## 4) React Query Provider 세팅

`src/app/providers/QueryProvider.tsx`

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactNode } from "react";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 1,
      refetchOnWindowFocus: false
    }
  }
});

export function QueryProvider({ children }: { children: ReactNode }) {
  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;
}
```

---

## 5) styled-components: GlobalStyle + Theme + 타입 보강

### 5-1) theme 정의

`src/shared/styles/theme.ts`

```ts
export const theme = {
  colors: {
    bg: "#0b0d12",
    surface: "#121621",
    text: "#e9ecf1",
    subText: "#aab2c0",
    primary: "#FEDA3E" // 사용자님 테마 컬러 기억 반영
  }
} as const;

export type AppTheme = typeof theme;
```

### 5-2) styled-components Theme 타입 연결

`src/styled.d.ts` (프로젝트 루트 또는 src 아래)

```ts
import "styled-components";
import type { AppTheme } from "./shared/styles/theme";

declare module "styled-components" {
  export interface DefaultTheme extends AppTheme {}
}
```

### 5-3) GlobalStyle

`src/shared/styles/GlobalStyle.ts`

```ts
import { createGlobalStyle } from "styled-components";

export const GlobalStyle = createGlobalStyle`
  *, *::before, *::after { box-sizing: border-box; }
  html, body { height: 100%; }
  body {
    margin: 0;
    background: ${({ theme }) => theme.colors.bg};
    color: ${({ theme }) => theme.colors.text};
    font-family: system-ui, -apple-system, Segoe UI, Roboto, "Noto Sans KR", sans-serif;
  }
  a { color: inherit; text-decoration: none; }
  button { font: inherit; }
`;
```

---

## 6) 라우터 + 레이아웃 기본 골격

`src/app/router.tsx`

```tsx
import { createBrowserRouter } from "react-router-dom";
import { RootLayout } from "./layouts/RootLayout";
import { HomePage } from "../pages/HomePage";
import { NotFoundPage } from "../pages/NotFoundPage";

export const router = createBrowserRouter([
  {
    element: <RootLayout />,
    children: [
      { path: "/", element: <HomePage /> }
    ],
    errorElement: <NotFoundPage />
  }
]);
```

`src/app/layouts/RootLayout.tsx`

```tsx
import { Outlet } from "react-router-dom";
import styled from "styled-components";

const Shell = styled.div`
  min-height: 100vh;
  display: flex;
  flex-direction: column;
`;

export function RootLayout() {
  return (
    <Shell>
      {/* Header 자리 */}
      <Outlet />
    </Shell>
  );
}
```

`src/pages/HomePage.tsx`

```tsx
import styled from "styled-components";

const Wrap = styled.main`
  padding: 24px;
`;

export function HomePage() {
  return (
    <Wrap>
      <h1>Home</h1>
      <p>Vite + React + TS + styled-components baseline</p>
    </Wrap>
  );
}
```

`src/pages/NotFoundPage.tsx`

```tsx
export function NotFoundPage() {
  return <div style={{ padding: 24 }}>Not Found</div>;
}
```

---

## 7) main.tsx에서 Provider 조립

`src/main.tsx`

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { RouterProvider } from "react-router-dom";
import { ThemeProvider } from "styled-components";
import { router } from "./app/router";
import { QueryProvider } from "./app/providers/QueryProvider";
import { GlobalStyle } from "./shared/styles/GlobalStyle";
import { theme } from "./shared/styles/theme";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <ThemeProvider theme={theme}>
      <GlobalStyle />
      <QueryProvider>
        <RouterProvider router={router} />
      </QueryProvider>
    </ThemeProvider>
  </React.StrictMode>
);
```

---

## 8) Husky + lint-staged 연결 (지금 package.json에 이미 준비됨)

`package.json`에 아래를 추가하세요.

```json
{
  "lint-staged": {
    "*.{ts,tsx,js,jsx}": ["eslint --fix", "prettier -w"],
    "*.{json,md,yml,yaml}": ["prettier -w"]
  }
}
```

그리고 husky 훅 생성:

```bash
npx husky add .husky/pre-commit "npx lint-staged"
```

---

## 9) (중요) 지금 패키지에서 “중복 상태관리” 정리 권장

현재 `@tanstack/react-query`가 있는데 `redux/react-redux`도 같이 있습니다.

- API 데이터는 **React Query**
    
- 전역 UI 상태가 정말 필요할 때만 zustand/redux
    

지금은 초기 구축 단계라면 **redux를 빼는 게 보통 더 깔끔**합니다. (남겨도 되지만, 팀 규칙 없으면 복잡도만 늘어나는 경우가 많습니다)

---

## 10) 추천 디렉토리 생성 한 번에

최소 디렉토리:

```
src/
  app/
    layouts/
    providers/
  pages/
  shared/
    api/
    styles/
```

---

원하시면 다음 단계로 바로 “Spring Security(세션/JWT) 전제”에 맞춰서

- `http.ts`에 **토큰 주입/리프레시 처리**
    
- 공통 에러(401/403/500) UI 처리 패턴
    
- `entities` 단위 API+Query 훅 템플릿(예: `useGetMe`, `useLogin`)  
    까지 한 세트로 드릴게요.
    

지금 백엔드 인증 방식이 **세션/쿠키 기반**인가요, 아니면 **JWT(Authorization 헤더)** 기반인가요? (둘 다 기준 코드가 달라서요)