# Hydration

**하이드레이션**(Hydration)은 프론트엔드 개발에서 서버사이드 렌더링(SSR, Server-Side Rendering)과 관련된 중요한 개념이다. 서버사이드 렌더링은 웹 페이지를 서버에서 미리 렌더링하여 클라이언트에게 HTML을 전달하는 방식이다. 이때 클라이언트 측에서 자바스크립트를 통해 추가적인 기능(예: 이벤트 처리, 상태 관리 등)을 활성화하는 과정을 **하이드레이션**이라고 한다.

> 서버가 HTML를 만들어서 먼저 화면을 보여주고,  
> 클라이언트에서 HTML에 React가 붙어서(hydrate) 클릭/상태/이벤트가 동작하도록 만드는 것이다.

- render : 화면을 그리는 것
- hydrate : 이미 그려진 화면에 이벤트/상태 로직을 연결해서 살아 있게 만드는 것

## 사용 이점

- 빠른 초기 로드
- 동적 상호작용
- SEO (검색 엔진 최적화)

## 동작 방식

### 서버사이드 렌더링 (SSR)
    
- 서버에서 미리 렌더링된 HTML이 클라이언트에게 전달됨.
- 이 HTML은 이미 완성된 구조를 가지고 있으나, 자바스크립트 로직은 아직 적용되지 않은 상태임.

### 하이드레이션
    
- 클라이언트 측에서 자바스크립트가 로드됨.
- 자바스크립트는 기존의 HTML과 연결되어 이벤트 리스너, 상태 관리, 기타 동적 기능을 활성화함.
- 이 과정을 통해 서버에서 받은 정적인 HTML이 클라이언트 측에서 동적인 웹 애플리케이션으로 변환됨.

## 예시 : React

### 1. 서버가 HTML을 내려줌

```html
<!-- index.html (서버가 만들어서 내려준 HTML이라고 가정) -->
<!doctype html>
<html>
  <body>
    <div id="root">
      <!-- 서버가 미리 렌더링해둔 마크업 -->
      <button id="btn">Count: 0</button>
    </div>

    <!-- 클라이언트 JS 번들 -->
    <script type="module" src="/client.tsx"></script>
  </body>
</html>
```
### 2. 클라이언트에서 hydration 수행

```ts
// client.tsx
import React, { useState } from "react";
import { hydrateRoot } from "react-dom/client";

function App() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount((c) => c + 1)}>
      Count: {count}
    </button>
  );
}

// 핵심: "이미 존재하는 #root HTML"에 React를 붙인다 (hydrate)
hydrateRoot(document.getElementById("root")!, <App />);
```

- React는 `#root` 안에 이미 있는 DOM(서버 HTML)을 “다시 만들지 않고”
- 그 DOM이 React가 만든 결과라고 “인식”한 뒤
- `onClick` 같은 이벤트/상태 로직을 연결해서 **인터랙티브**하게 만듭니다.

### 3. 비교: hydrate가 아니라 createRoot로 렌더하면?

```ts
import React from "react";  
import { createRoot } from "react-dom/client";  
  
createRoot(document.getElementById("root")!).render(<div>새로 렌더</div>);
```

- `createRoot().render()`는 보통 **클라이언트에서 DOM을 새로 만들며**
- 서버 HTML이 있었더라도 **갈아끼우는** 느낌이 됩니다.
- 반면 `hydrateRoot()`는 **서버 HTML을 최대한 재사용**하면서 연결만 합니다.

# 출처

[gpt](https://chatgpt.com/share/699d4d7e-2ffc-8003-88da-362c9c6b47e3)  
[티스토리](https://disco-biscuit.tistory.com/115)