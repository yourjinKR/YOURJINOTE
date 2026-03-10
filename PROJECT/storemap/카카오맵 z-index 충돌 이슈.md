
# 카카오맵 Marker와 CustomOverlay z-index 충돌 트러블슈팅

## 1. 문제 상황

카카오맵 API를 사용해 지도에 `Marker`와 `CustomOverlay`를 함께 표시하던 중, 다음과 같은 문제가 발생했습니다.

- 오버레이 문구가 마커보다 위에 보여야 했다.
    
- 하지만 오버레이의 `zIndex`를 높이면 마커 클릭이 되지 않았다.
    
- 반대로 마커 클릭이 가능하도록 조정하면, 마커가 오버레이를 가려 문구가 정상적으로 보이지 않았다.
    

즉,  
**“오버레이는 마커보다 위에 보여야 하지만, 마커 클릭은 유지되어야 하는”**  
상충되는 요구사항을 해결해야 했다.

---

## 2. 초기 가설

처음에는 일반적인 DOM/CSS 문제라고 판단했다.  
그래서 다음과 같은 방향으로 접근했다.

- `Marker`의 `zIndex`를 낮추고 `CustomOverlay`의 `zIndex`를 높인다.
    
- 오버레이 내부 `.customoverlay`, `.title`의 `z-index`를 세분화한다.
    
- `pointer-events: none / auto`를 조합해 클릭 이벤트를 분리한다.
    
- `bottom`, `transform`, `yAnchor` 등을 조정해 마커와 오버레이의 시각적 겹침을 해소한다.
    

---

## 3. 시도한 방법과 결과

### (1) CSS z-index 조정

```css
.customoverlay {
    position: relative;
    z-index: 8;
}

.customoverlay .title {
    position: relative;
    z-index: 9999;
}
```

**결과**

- 개발자 도구 상에서는 `z-index`가 정상적으로 반영되었다.
    
- 하지만 실제 화면에서는 여전히 마커가 오버레이를 가렸다.
    

**판단**

- 단순 CSS stacking 문제가 아니라, 카카오맵 내부 렌더링 구조 차이일 가능성을 의심했다.
    

---

### (2) pointer-events 조정

```css
.customoverlay {
    pointer-events: none;
}

.customoverlay .title {
    pointer-events: auto;
}
```

**결과**

- 일부 클릭 충돌은 완화할 수 있었지만,
    
- 오버레이가 마커 위에 떠 있는 상태에서는 여전히 마커 클릭이 막히는 경우가 발생했다.
    

**판단**

- 시각적 우선순위와 클릭 우선순위를 CSS만으로 동시에 만족시키기 어려웠다.
    

---

### (3) 위치 이동 (`bottom`, `transform`, `yAnchor`)

```css
.customoverlay {
    bottom: 50px;
}
```

또는

```css
.customoverlay {
    transform: translateY(-80px);
}
```

**결과**

- 오버레이를 위로 띄우는 것은 가능했지만,
    
- 근본적인 z-index 충돌 자체를 해결하지는 못했다.
    
- 특히 `yAnchor`는 기대만큼 영향을 주지 않았다.
    

**판단**

- 오버레이를 “위치상 띄우는 것”과 “레이어 우선순위 해결”은 별개의 문제였다.
    

---

### (4) 카카오맵이 생성한 부모 div를 직접 조작

오버레이 생성 후 카카오맵이 만든 부모 div를 찾아 직접 `z-index`나 `pointer-events`를 조정하려고 했다.

예를 들어:

```javascript
function lowerOverlayShellZIndex() {
    document.querySelectorAll('.customoverlay').forEach(function(overlay) {
        var container = overlay.parentElement;
        if (container && container.style) {
            container.style.zIndex = '1';
        }
    });
}
```

또는 클릭 방해를 줄이기 위해:

```javascript
function preventOverlayClickBlock() {
    document.querySelectorAll('.customoverlay').forEach(function(overlay) {
        var container = overlay.parentElement;
        if (container && container.style) {
            container.style.pointerEvents = 'none';
        }
        overlay.style.pointerEvents = 'auto';
    });
}
```

**결과**

- 부모 div의 `z-index`를 낮추면 내부 자식도 함께 영향받아 오버레이가 다시 가려졌다.
    
- 부모 div의 클릭만 막는 방식은 일부 효과가 있었지만, 구조적 한계를 완전히 해결하지는 못했다.
    

**추가 이슈**

- Eclipse에서는 `optional chaining(?.)` 문법 때문에 에러가 발생해,  
    ES5 호환 문법으로 수정해야 했다.
    

---

## 4. 원인 분석

최종적으로 확인한 핵심 원인은 **카카오맵 API의 렌더링 구조 차이**였다.

### 구조적 차이

- `Marker`는 일반 HTML DOM이 아니라, 카카오맵 내부의 별도 레이어(이미지/캔버스 기반)에 렌더링된다.
    
- `CustomOverlay`는 DOM 기반 HTML 요소로 삽입된다.
    

즉, 두 요소는 **같은 stacking context 안에서 경쟁하는 것이 아니기 때문에**,  
브라우저 CSS의 `z-index`만으로 완전히 제어할 수 없다.

### 추가적으로 확인한 CSS 개념

부모 요소가 stacking context를 형성하면,  
자식 요소가 아무리 큰 `z-index`를 가져도 부모보다 앞선 다른 레이어를 뚫고 나갈 수 없다.

이 때문에:

- 부모 div `z-index: 1`
    
- 자식 `.customoverlay` `z-index: 9999`
    

처럼 설정해도, 자식은 부모의 stacking context 밖으로 나갈 수 없었다.

---

## 5. 결론

이 문제는 단순한 CSS 실수나 이벤트 처리 오류가 아니라,  
**카카오맵 API의 Marker와 CustomOverlay가 서로 다른 렌더링 계층에 존재하는 구조적 한계**에서 발생한 이슈였다.

즉,

- `Marker`와 `CustomOverlay`를 완전히 독립적으로 유지하면서
    
- 오버레이를 항상 마커보다 위에 보이게 하고
    
- 동시에 마커 클릭도 100% 보장하는 것
    

은 **기본 제공 구조만으로는 완전한 해결이 어렵다**는 결론에 도달했다.

---

## 6. 배운 점

이번 이슈를 통해 단순히 “CSS를 잘 다루는 것”보다 더 중요한 점을 배웠다.

### 1) UI 문제는 브라우저 CSS만의 문제가 아닐 수 있다

처음에는 단순한 `z-index` 문제라고 생각했지만, 실제로는 외부 API의 렌더링 구조를 이해해야 했다.

### 2) 스태킹 컨텍스트를 실무적으로 이해하게 되었다

부모와 자식의 `z-index` 관계, 그리고 서로 다른 렌더링 계층에서는 CSS가 기대대로 작동하지 않을 수 있음을 확인했다.

### 3) “무조건 해결”보다 “한계를 정확히 파악하는 것”도 중요하다

실무에서는 모든 문제가 코드 몇 줄로 해결되지 않는다.  
중요한 것은 여러 가설을 검증하고, 어디까지가 애플리케이션 코드의 문제이고 어디부터가 외부 라이브러리의 구조적 한계인지 구분하는 것이다.

---

## 7. 포트폴리오용 한 줄 정리

**카카오맵 API 사용 중 Marker와 CustomOverlay 간 z-index 및 클릭 충돌 문제를 분석하며, 단순 CSS 이슈가 아닌 외부 지도 API의 렌더링 구조 차이와 stacking context 한계를 파악하고 우회 전략을 검토한 경험**

---

## 8. 자소서용 표현 예시

### 버전 1. 문제 해결형

카카오맵 API를 활용한 지도 기능 구현 과정에서 Marker와 CustomOverlay 간의 z-index 및 클릭 충돌 문제를 겪었습니다. 초기에는 CSS 조정만으로 해결할 수 있다고 판단했지만, 다양한 실험과 디버깅을 통해 해당 이슈가 브라우저 레벨이 아닌 지도 API의 렌더링 구조 차이에서 비롯된 문제임을 파악했습니다. 이를 통해 단순 구현을 넘어서 외부 라이브러리의 동작 원리를 분석하고, 기술적 한계까지 구분해 문제를 해석하는 역량을 키웠습니다.

### 버전 2. 탐구형

지도 UI 구현 중 오버레이와 마커의 레이어 충돌 문제를 해결하기 위해 `z-index`, `pointer-events`, `yAnchor`, DOM 후처리 등 여러 방법을 검증했습니다. 그 과정에서 CSS stacking context와 카카오맵 API의 내부 렌더링 구조를 깊이 이해하게 되었고, 결국 문제의 본질이 애플리케이션 코드가 아닌 외부 API의 구조적 특성에 있음을 확인했습니다. 이 경험을 통해 표면적인 증상만 보는 것이 아니라, 원인을 구조적으로 분석하는 습관을 갖게 되었습니다.

### 버전 3. 협업/실무형

지도 기능 개발 과정에서 발생한 UI 충돌 문제를 해결하기 위해 다양한 가설을 세우고 직접 검증했습니다. 단순 스타일 수정에 그치지 않고, DOM 구조 분석과 이벤트 처리 방식, 외부 API의 렌더링 계층까지 추적하며 원인을 좁혀나갔습니다. 비록 라이브러리의 구조적 한계로 완전한 해결에는 제약이 있었지만, 문제를 끝까지 추적하며 기술적 한계를 명확히 문서화한 경험은 실무적인 문제 해결 역량을 키우는 계기가 되었습니다.
