# 요구사항

- 구현 코드를 보고 포트폴리오 양식에 맞춰 구현 기능 부분을 작성 바람
- 구현한 기능을 "문제 → 해결 → 결과" 를 중점으로 작성
- 해당 프로젝트는 주로 프론트 업무를 수행했음


# 포트폴리오 양식 (타 프로젝트 예시)

### 1. OpenAI API 기반 운동 루틴 추천 및 API 요청 로그 분석 시스템

- OpenAI API를 활용해 **사용자 프로필 및 운동 기록 기반 맞춤 운동 루틴 추천 기능**을 구현하였으며,
    
    DB에 존재하는 운동 데이터만 추천하도록 제약을 두어 **AI 응답의 일관성과 신뢰성**을 확보함
    
- AI 요청/응답 전문, 사용 모델, 토큰 사용량, 처리 시간 등을 로그로 저장하여
    
    **API 호출 이력 추적 및 비용·성능 분석이 가능한 구조**를 설계함
    
- AI 응답 구조 변경(버전 차이) 및 비정상 응답에 대비하여
    
    **서비스 레벨에서 파싱 및 검증 로직을 분리**하고, 실패 사유를 상태값으로 기록하여
    
    운영 중 발생하는 문제를 빠르게 파악할 수 있도록 개선함
    
- 구독 사용자에 한해 AI 추천 및 피드백 기능을 제공하여
    
    **결제 시스템과 AI 기능을 연계한 서비스 흐름**을 구성함
    

### 2. 구독·정기결제 시스템 및 정기결제 스케줄링

- PortOne(BillingKey) 기반 결제 시스템을 도입하여
    
    **결제수단 등록 → 단건 결제 → 정기결제 예약**으로 이어지는 구독 결제 플로우를 구현함
    
- 정기결제 예약 생성·변경·취소 기능을 설계하고,
    
    예약 시간 유효성 검증 및 중복 예약 방지 로직을 통해 **안정적인 스케줄 관리**를 구현함
    
- 예약 결제 상태를 주기적으로 확인하는 **배치 모니터링 시스템**을 구축하여
    
    외부 결제 API 상태와 내부 DB 상태를 동기화하고,
    
    결제 성공 시 다음 결제를 자동 예약하는 **구독 자동화 로직**을 구현함
    
- 결제 상태 업데이트 시 조건부 트랜잭션과 짧은 트랜잭션 범위를 적용하여
    
    **중복 처리 및 동시성 문제를 방지**하고,
    
    API 호출 제한·타임아웃·예외 처리로 운영 안정성을 고려한 구조를 설계함

# 구현 코드

### map.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
<div class="main-map">
	<div class="map" id="full" style="width: 100%; height: 100vh;"></div>
	<!-- 리스트 사이드바 -->
	<div class="side-bar show" id="list">
		<div id="content">
			<div class="header" id="list">
			  <div class="logo">
			    <a href="/">Eventore</a>
			  </div>
			
			  <div class="inner">
			    <form class="form" id="map">
			      <input type="text" id="keyword" autocomplete="off" placeholder="장소 및 이벤트 검색">
			      <div class="autocomplete"><ul></ul></div>
			    </form>
			
			    <div id="filter">
			      <button class="mapBtn select" id="united-mode">모두</button>
			      <button class="mapBtn" id="event-mode">이벤트</button>
			      <button class="mapBtn" id="store-mode">점포</button>
			    </div>
			  </div>
			</div>
			<div id="body">
				<jsp:include page="../modal/storeListModal.jsp"/>
				<jsp:include page="../modal/eventListModal.jsp"/>
				<div class="search-fail" id="united">
					<div id="fail-content">검색 결과가 존재하지 않습니다.</div>
				</div>
				<!-- <button class="mapBtn" id="search-more">결과값 더보기</button> -->				
			</div>
		</div>
	</div>

	<!-- 맵 관련 편의기능 내비게이션 -->
	<div id="map-navigation">
		<!-- <button class="mapBtn" id="custom-position">현위치 재설정</button> -->
		<button class="mapBtn" id="panto-current">
			<i class="fa-solid fa-location-crosshairs"></i>
		</button>
	</div>
	
	<!-- 사이드바 컨트롤러 -->
	<div class="side-bar" id="toggle-box">
		<button class="mapBtn" id="toggle">&gt;</button>			
	</div>
	
	<!-- 통합 모달 -->
	<div class="side-bar" id="united">
		<div class="modal-header">
			<button class="mapBtn" id="close-united">X</button>
		</div>
		<div class="modal-content" id="united">
			
		</div> 
	</div>
</div>
```

### map.js

```javascript
("map load");

// 마커 아이콘 설정 kakao.maps.MarkerImage(src, size[, options])
// ================== 마커 src ==================
let basicMarkerSrc = 'https://cdn3.iconfinder.com/data/icons/google-material-design-icons/48/ic_location_on_48px-256.png';
let storeMarkerSrc = 'https://res.cloudinary.com/dbdkdnohv/image/upload/v1748482122/ChatGPT_Image_2025%EB%85%84_5%EC%9B%94_29%EC%9D%BC_%EC%98%A4%EC%A0%84_10_28_30_wglij6.png';
let eventMarkerSrc = 'https://res.cloudinary.com/dbdkdnohv/image/upload/v1748481109/ChatGPT_Image_2025%EB%85%84_5%EC%9B%94_29%EC%9D%BC_%EC%98%A4%EC%A0%84_09_51_43_hhnjza.png';

// ================== 마커 크기 ==================
const MARKER_WIDTH = 32, // 기본 마커의 너비
    MARKER_HEIGHT = 35; // 기본 마커의 높이
const CLICKED_WIDTH = 40, // 클릭 마커의 너비
    CLICKED_HEIGHT = 47; // 클릭 마커의 높이
const markerSize = new kakao.maps.Size(MARKER_WIDTH, MARKER_HEIGHT), // 기본, 클릭 마커의 크기 
    clickedMarkerSize = new kakao.maps.Size(CLICKED_WIDTH, CLICKED_HEIGHT); // 오버 마커의 크기

// ================== 마커 option ================== 
let markerOption = { offset: new kakao.maps.Point(16, 34), alt: "마커 이미지 예제", shape: "poly", coords: "1,20,1,9,5,2,10,0,21,0,27,3,30,9,30,20,17,33,14,33" };
let clickedMarkerOption = { offset: new kakao.maps.Point(20, 46), alt: "클릭 마커 이미지 예제", shape: "poly", coords: "1,20,1,9,5,2,10,0,21,0,27,3,30,9,30,20,17,33,14,33" };

// 마커 아이콘
let testIcon = new kakao.maps.MarkerImage(basicMarkerSrc, markerSize, markerOption);
let clickedIcon = new kakao.maps.MarkerImage(basicMarkerSrc, clickedMarkerSize, clickedMarkerOption);
// 점포 아이콘
let storeIcon = new kakao.maps.MarkerImage(storeMarkerSrc, markerSize, markerOption);
let storeClickedIcon = new kakao.maps.MarkerImage(storeMarkerSrc, clickedMarkerSize, clickedMarkerOption);
// 이벤트 아이콘
let eventIcon = new kakao.maps.MarkerImage(eventMarkerSrc, markerSize, markerOption);
let eventClickedIcon = new kakao.maps.MarkerImage(eventMarkerSrc, clickedMarkerSize, clickedMarkerOption);
/** 아이콘을 저장하는 객체 */
const iconObj = {
    testIcon : testIcon,
    clickedIcon : clickedIcon,
    storeIcon : storeIcon,
    storeClickedIcon : storeClickedIcon,
    eventIcon : eventIcon, 
    eventClickedIcon : eventClickedIcon
}

// 클릭하여 강조된 마커
let selectedMarker = { marker: null, type: null };
/** 지도 클릭 이벤트 마커 */
let clickMarker = null;

// 지도에 표시된 마커 객체를 가지고 있을 배열
let markerList = [];
/** store vo 리스트 */
let storeVOList = [];
/** event vo 리스트 */
let eventVOList = [];
/** vo 리스트를 관리하는 객체 */
let VOListObj = {
    store : storeVOList,
    event : eventVOList
}
/** 장소 정보를 담는 리스트 */
let placeList = [];
/** 장소 정보 (음식점 제외) */
let placeInfo = {};
/** 지하철 정보 */
let subwayInfo = {};

// 출력 위치 확인
let storeUL = null;

// 스토어 리스트 사이드바 컨트롤
let listSideBar = null;
// 스토어 리스트의 검색 결과 div
let searchResult;
/** true : 사이드바가 열린 상태 */
let viewSideBarCheck = false;
let viewSideBar = null;
/** 토글 버튼 박스 */
let toggleBtnBox = null;
/** 토글 버튼 */
let toggleBtn = null;
// 스토어 리스트바 모달
let storeListModal = null;
// 이벤트 리스트바 모달
let eventListModal = null;
/** 점포 검색 실패 모달 */
let storeFailModal = null;
/** 이벤트 검색 실패 모달 */
let eventFailModal = null;
/** 통합 검색 실패 모달 */
let unitedFailModal = null;
// 검색창
let mapSearchForm = null;
let keywordInput;
// 자동완성
let autoCompleteBox = null;
let autoSearchUL = null;
/** 자동완성 재입력 방지 */
let suppressAutocomplete = false;
let selectedIndex = -1;
/** 검색어 조건 */
let searchCondition = { lat: null, lng: null, kilometer: null, level: null, code: null, amount: null, keyword: null, store_idx: null, event_idx: null };

// 기본 위도 경도 설정 (솔데스크 강남점)
let basicLat = 37.505410898990775;
let basicLng = 127.02676741751233;

// 현재 지도의 중심
let centerLatLng;
// 현재 지도의 행정구역 (중심 기준)
let centerLoc = {}

// 지도 변수 생성
let basicMap;
// 지도 타입
let mapType;

/** 통합 검색 */
let unitedMapMode = true;
/** 이벤트 검색 */
let eventMapMode = false;
/** 점포 검색  */
let storeMapMode = false;
/** 현위치 커스텀 설정 모드 */
let customPositionMode = false;
/** 현위치 커스텀 설정 오버레이 */
let positionOverlay = null;

// 페이지 로드 후 script 실행
document.addEventListener("DOMContentLoaded", () => {
    const CSS_PATH = '/resources/css/map.css';
    let linkEle = document.createElement('link');
    linkEle.rel = 'stylesheet';
    linkEle.href = CSS_PATH;
    document.head.appendChild(linkEle);

    /** 로컬 스토리지 기반 경도위도 값 */
    let currentPosition = getPositionData();

    // 지도 기본 설정값
    let basicOption = {};
    if (currentPosition) {
        (currentPosition);

        currentLat = currentPosition.lat;
        currentLng = currentPosition.lng;
        basicOption = { center: new kakao.maps.LatLng(currentLat, currentLng), level: 3 };

        clickMarker = new kakao.maps.Marker({
            position: new kakao.maps.LatLng(currentLat, currentLng)
        });

    } else {
        ('없음', currentPosition);
        basicOption = { center: new kakao.maps.LatLng(basicLat, basicLng), level: 1 };
        currentLat = basicLat;
        currentLng = basicLng;

        clickMarker = new kakao.maps.Marker({
            position: new kakao.maps.LatLng(basicLat, basicLng)
        });
    }

    // 클릭마커 클릭 이벤트 추가
    kakao.maps.event.addListener(clickMarker, 'click', function () {
        if (!customPositionMode && mapType === "full") {
            // alert("현위치를 수정하시겠습니까?");
            setMyCurrentPlace();
        }
    })

    // 제한할 범위 설정 (예: 대한민국 전체)
    const boundLimit = new kakao.maps.LatLngBounds(
        new kakao.maps.LatLng(33.0, 124.0),  // 남서쪽
        new kakao.maps.LatLng(39.5, 132.0)   // 북동쪽
    );

    // 기본 설정값으로 지도 생성
    let container = document.querySelector(".map");

    if (container != null) {
        basicMap = new kakao.maps.Map(container, basicOption);

        // 새로고침
        centerLatLng = basicMap.getCenter();
        loadAddrFromCoords(centerLatLng);

        // 클릭 마커 생성  
        clickMarker.setMap(basicMap);

        // 맵 id별 분기
        mapType = container.getAttribute("id");

        // 최대크기 지정
        basicMap.setMaxLevel(12);
    }

    // 검색창
    mapSearchForm = document.querySelector(".form#map");

    if (mapType === "full") {
        keywordInput = mapSearchForm.keyword;
    } else {
        mapSearchForm = document.querySelector(".search-bar");
        keywordInput = document.querySelector(".search-bar input[name='keyword']");
    }

    if (mapSearchForm != null) {
        autoCompleteBox = mapSearchForm.querySelector(".autocomplete");
    }
    if (autoCompleteBox != null) {
        autoSearchUL = autoCompleteBox.querySelector(".autocomplete ul");
    }

    // map.jsp
    if (mapType === "full") {
        // 지도 확대 축소를 제어할 수 있는  줌 컨트롤을 생성합니다
        let zoomControl = new kakao.maps.ZoomControl();
        basicMap.addControl(zoomControl, kakao.maps.ControlPosition.RIGHT);

        // 가게 상세보기 모달 사이드바 style로 변경
        let storeModal = document.querySelector("#modal");
        storeModal.classList.add("side-bar");
        storeModal.setAttribute("id", "store");
        // 닫기 버튼 표시
        let storeHeader = document.querySelectorAll(".modal-header")[0];
        storeHeader.style.display = 'block';

        // 이벤트 상세보기 모달 사이드바 style로 변경
        let eventModal = document.querySelector("#modal");
        eventModal.classList.add("side-bar");
        eventModal.setAttribute("id", "event");
        // 닫기 버튼 표시
        let eventHeader = document.querySelectorAll(".modal-header")[1];
        eventHeader.style.display = 'block';

        // 닫기 버튼 표시
        let unitedHeader = document.querySelectorAll(".modal-header")[2];
        unitedHeader.style.display = 'block';

        // 기본 상태
        searchCondition = {
            level: basicMap.getLevel(),
            lat: basicMap.getCenter().getLat(),
            lng: basicMap.getCenter().getLng(),
            code: centerLoc.code,
            // keyword : keyword,
            amount: 100,
            kilometer: setRadiusByLevel(basicMap.getLevel())
        }

        // ==================== 요소 선언 ====================
        // 검색바
        searchResult = document.querySelector(".search-result#store-list");
        // 스토어 리스트 사이드바
        listSideBar = document.querySelector(".side-bar#list");
        //  뷰 사이드바
        viewSideBarCheck = false;
        viewSideBar = document.querySelector(".side-bar#united");
        // 토글 버튼 박스
        toggleBtnBox = document.querySelector(".side-bar#toggle-box");
        // 토글 버튼
        toggleBtn = document.querySelector(".mapBtn#toggle");
        // 스토어 리스트 모달
        storeListModal = document.querySelector(".storeListModal");
        // 이벤트 리스트 모달
        eventListModal = document.querySelector(".eventListModal");
        // 점포 검색 실패 모달
        storeFailModal = document.querySelector(".search-fail#store");
        // 이벤트 검색 실패 모달
        eventFailModal = document.querySelector(".search-fail#event");
        // 통합 검색 실패 모달
        unitedFailModal = document.querySelector(".search-fail#united");

        // 검색 리스트 스타일 변경
        storeListModal.style.display = 'block';
        eventListModal.style.display = 'block';

        deleteAllEle();
        // 초기 검색 기능
        // 키워드 기반
        const initialKeyword = sessionStorage.getItem("initialKeyword");
        const initialStoreIDX = sessionStorage.getItem("store_idx");
        const initialEventIDX = sessionStorage.getItem("event_idx");
        const subCondition = JSON.parse(localStorage.getItem("search_data"));

        if (initialKeyword) {
            // input에도 값 넣기 (선택사항)
            keywordInput.value = initialKeyword;

            // 검색 조건 구성 후 실행
            mapSearchService(initialKeyword);
            keywordInput.value = '';

            // 재검색 방지: 세션 제거
            sessionStorage.removeItem("initialKeyword");
        }
        else if (initialStoreIDX) {
            // idx 기반 점포 찾기
            sessionStorage.removeItem("store_idx");
            showListSideBar();
            // store idx 기반 검색 비동기 함수 실행
            // let IDX = 302;
            as.getStoreByIdx(initialStoreIDX, function (data) {
                searchCondition.keyword = data.store_name;
                setSubKeyword();
                apply2storeMap([data]);
            });
        }
        else if (initialEventIDX) {
            // idx 기반 이벤트 찾기
            sessionStorage.removeItem("event_idx");
            showListSideBar();
            // event idx 기반 검색 비동기 함수 실행
            // let IDX = 302;
            as.getEventByIdx(initialEventIDX, function (data) {
                searchCondition.keyword = data.event_title;
                setSubKeyword();
                processAllEvents([data]);
            });
        }
        else if (subCondition != null) {
            if (subCondition.store_idx != null) {
                as.getStoreByIdx(subCondition.store_idx, function (data) {
                    apply2storeMap([data]);
                    semiToggle();
                })
            } else if (subCondition.event_idx != null) {
                as.getEventByIdx(subCondition.event_idx, function (data) {
                    processAllEvents([data]);
                })
            }
            else if (subCondition.keyword != null) {
                searchCondition = subCondition;
                basicMap.setLevel(searchCondition.level);
                mapSearchService(searchCondition.keyword);
            } else {
                as.getListNearest(searchCondition, 5, function (data) {
                    apply2storeMap(data);
                })
            }
        }
    }
    // storeModify.jsp (영업 위치 설정 지도)
    else if (mapType === "store-loc") {
        let f = document.forms[0];
        // 위치가 설정되지 않았을 경우
        if (f.lat.value != 0 && f.lng.value != 0) {
            let latlng = new kakao.maps.LatLng(f.lat.value, f.lng.value);
            // 지도 중심 화면 설정
            basicMap.setCenter(latlng);
            // 지도 중심 위치 설정
            basicLat = f.lat.value;
            basicLng = f.lng.value;

            clickMarker.setPosition(latlng);
        }
    }
    // 우편번호 입력 폼
    else if (mapType === "postcode") {
        const f = document.forms[0];
        let result = locationToCoords(f.event_location.value);
    }

    // 지도 관련 버튼 서비스
    document.querySelectorAll("button.mapBtn").forEach(btn => {
        btn.addEventListener("click", e => {
            e.preventDefault();
            let type = btn.getAttribute("id");
            // 지도 중심좌표 부드럽게 이동하기
            if (type === "panToTest") {
                panToLatLng(basicMap, basicLat, basicLng);
            }
            // 마커 리스트 비울때는 숨김 처리 후 리스트의 요소를 비워야 정상 작동
            else if (type === "markersClear") {
                hideMarkers(basicMap);
                clearMarkers();
            }
            // 사이드바 열기
            else if (type === "showListSideBar") {
                showListSideBar();
            }
            // 사이드바 열기
            else if (type === "hideListSideBar") {
                hideListSideBar();
            }
            // 토글 버튼
            else if (type === "toggle") {
                toggleListSideBar();
            }
            // 사이드바 닫기 버튼
            else if (type === "close-store" || type === "close-event" || type === "close-united") {
                viewSideBarCheck = false;
                semiToggle();
                hideviewSideBar();
            }
            // 우편번호 찾기 버튼
            else if (type === "search-postcode") {
                pcodeService(basicMap);
            }
            // 영업 위치설정
            else if (type === "store-loc") {

            }
            // 현위치 이동
            else if (type === "panto-current") {
                handleCurrentLocationClick(basicMap, currentLat, currentLng, mapType);
            }
            // 검색
            else if (type === "search") {
                let f = document.querySelector(".form#map");
                let keyword = f.keyword.value.trim();

                mapSearchService(keyword);
            }
            else if (type === "event-mode") {
                ctrlMapMode("event");
            }
            else if (type === "store-mode") {
                ctrlMapMode("store");
            }
            else if (type === "united-mode") {
                ctrlMapMode("united");
            }
            else if (type === "custom-position") {
                setMyCurrentPlace();
            }
        });
    });

    // 현위치 커스텀 설정 오버레이를 생성합니다
    positionOverlay = new kakao.maps.CustomOverlay({
        content: `
                <div class="customoverlay">
                    <span class="title">현위치 설정</span>
                    <button onclick="getMyCurrentPlace()">변경</button>
                    <button onclick="cancelMyCurrentPlace()">취소</button>
                </div>`,
        yAnchor: 1,
        zIndex: 3
    });

    if (basicMap != null) {
        /** 지도 새로고침 */
        basicMap.relayout();

        // 지도 클릭 이벤트 (경도 위도 출력)
        kakao.maps.event.addListener(basicMap, 'click', function (mouseEvent) {

            // 클릭한 위도, 경도 정보
            let latlng = mouseEvent.latLng;

            if (mapType === "full" && customPositionMode) {
                clickMarker.setPosition(latlng);
                positionOverlay.setPosition(latlng);
            }

            if (mapType === "store-loc") {
                clickMarker.setPosition(latlng);
                let f = document.querySelector("#store-modify");
                if (f) {
                    f.lat.value = latlng.getLat();
                    f.lng.value = latlng.getLng();
                    initRCodeFromCoords(latlng, f);
                    initDetailAddrFromCoords(latlng, f);
                } else {
                    
                }
            }
        });

        // 줌 이벤트
        kakao.maps.event.addListener(basicMap, 'zoom_changed', function () {
            const level = basicMap.getLevel();

            if (level > 3) {
                hideOverlay(storeVOList);
                hideOverlay(eventVOList);
            } else {
                if (storeMapMode || unitedMapMode) {
                    showOverlay(basicMap, storeVOList);
                }
                if (eventMapMode || unitedMapMode) {
                    showOverlay(basicMap, eventVOList);
                }
            }

            if (clickedOverlay != null) {
                clickedOverlay.setMap(basicMap);
            }
            preventOverlayClickBlock();
        });

        // 지도 이동이 완료되었을 발생하는 이벤트
        kakao.maps.event.addListener(basicMap, 'dragend', function () {
            const center = basicMap.getCenter();

            // 중심 좌표가 제한된 범위를 벗어나면
            if (!boundLimit.contain(center)) {
                // 가장 가까운 지점으로 중심을 재조정
                const lat = Math.min(Math.max(center.getLat(), boundLimit.getSouthWest().getLat()), boundLimit.getNorthEast().getLat());
                const lng = Math.min(Math.max(center.getLng(), boundLimit.getSouthWest().getLng()), boundLimit.getNorthEast().getLng());

                const newCenter = new kakao.maps.LatLng(lat, lng);
                basicMap.setCenter(newCenter);
            }
        });
    }



    // ====================== 검색어 자동완성 ====================== 
    if (keywordInput != null) {

        // 방향키로 선택 중인 항목의 인덱스
        selectedIndex = -1;
        /** 자동완성을 취소했을때 돌아갈 키워드 */
        let orgKeyword = null;
    
        let suggestionList = []; // 현재 렌더링 중인 추천 리스트
    
        // 검색창이 포커스될때
        keywordInput.addEventListener("focus", e => {
            // <li> 자식 요소가 하나라도 있는 경우에만 display
            if (autoCompleteBox.querySelectorAll("li").length > 0) {
                autoCompleteBox.style.display = "block";
            } else {
                as.getListNearest(searchCondition, 10 , (data) => {
                    updateSuggestionList(data, 'store');
                })
            }
        });
    
        // 검색창에 값 입력
        keywordInput.addEventListener("input", e => {
    
            resetAutocomplete();
    
            const keyword = keywordInput.value.trim();
    
            if (keyword.length === 0) {
                return;
            }
    
            let lat;
            let lng;
            // 검색어 미리보기는 전국 단위로 확인
            let kilometer = 999;
            // 검색어 조건 설정
            // map.jsp에서
            if (basicMap != null && mapType === "full") {
                lat = basicMap.getCenter().getLat();
                lng = basicMap.getCenter().getLng();
                // kilometer = setRadiusByLevel(basicMap.getLevel());
            } else {
                lat = currentLat;
                lng = currentLng;
                // kilometer = setRadiusByLevel(5);
            }
    
            // 점포 정보 불러오기 (서버에 비동기 요청)
            fetch(`/modal/list/keyword.json`, {
                method: 'POST',
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify({ keyword: keyword, lat: lat, lng: lng, amount: 0, kilometer: kilometer })
            })
                .then(response => response.json())
                .then(data => {
    
                    // 최대 5개까지만 표시
                    // const suggestionList = data.slice(0, 3);
                    updateSuggestionList(data, 'store', keyword);
                })
                .catch(err => {
                    console.error("자동완성 fetch 실패", err);
                    // resetAutocomplete();
                });
    
            // 이벤트 정보 불러오기 (서버에 비동기 요청)
            fetch("/event/eventFilter/keyword", {
                method: 'POST',
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify({ keyword: keyword, lat: lat, lng: lng, amount: 0 })
            })
                .then(response => response.json())
                .then(data => {
                    // (data);
    
                    // processAllEvents(data, "autoComplete");
    
                    // 최대 5개까지만 표시
                    // const suggestionList = data.slice(0, 3);
                    updateSuggestionList(data, 'event', keyword);
                })
                .catch(err => {
                    console.error("자동완성 fetch 실패", err);
                    // resetAutocomplete();
                });
    
    
        });
    
        // 자동완성 리스트 닫기
        document.addEventListener("click", function (e) {
            if (!mapSearchForm.contains(e.target)) {
                // ('다른 곳을 클릭했음으로 닫음');
                hideAutocomplete();
            }
        });
    
        // 자동완성 검색창 키보드 방향키 지원
        keywordInput.addEventListener("keydown", e => {
            //const items = autoSearchUL.querySelectorAll("li");
            const items = Array.from(autoSearchUL.querySelectorAll("li")).filter(li => {
                return window.getComputedStyle(li).display === "block";
            });
            // if (items.length === 0) return;
    
            switch (e.keyCode) {
                case 38: // 위
                    e.preventDefault();
                    if (selectedIndex > 0 && autoCompleteBox.style.display === 'block') {
                        selectedIndex--;
                        updateActiveItem(items);
                        // input에 값 적용
                        keywordInput.value = items[selectedIndex].dataset.value;
                    }
                    // 자동완성 리스트에 벗어나기
                    else if (selectedIndex == 0) {
                        if (orgKeyword != null) {
                            keywordInput.value = orgKeyword;
                        }
                        // resetAutocomplete();
                        hideAutocomplete();
                    }
                    break;
    
                case 40: // 아래
                    e.preventDefault();
                    if (selectedIndex < items.length - 1 && autoCompleteBox.style.display === 'block') {
                        // 자동완성 첫 진입시 기존의 키워드를 기억
                        if (selectedIndex == -1) {
                            orgKeyword = keywordInput.value;
                        }
                        selectedIndex++;
                        updateActiveItem(items);
                        // input에 값 적용
                        keywordInput.value = items[selectedIndex].dataset.value;
                    }
                    break;
    
                case 13: // Enter
                    // 검색 키워드 기억
                    e.preventDefault();
                    deleteAllEle();
    
                    suppressAutocomplete = true;
    
                    // 자동완성 검색어를 선택했을때
                    if (selectedIndex >= 0 && selectedIndex < items.length) {
                        const selectedItem = items[selectedIndex];
                        const value = selectedItem.dataset.value; // 정확한 값 추출
                        const type = selectedItem.getAttribute("type"); // 강사님께 질문
                        const idx = selectedItem.getAttribute("idx");
    
                        keywordInput.value = value;

                        // 주변 검색
                        if (value === '내 주변') {
                            if (mapType === 'full') {
                                mapSearchService(value);
                            } else {
                                sessionStorage.setItem("initialKeyword", value);
                                location.href = "/store/map";
                            }
                            return;
                        }

                        // resetAutocomplete();
                        if (mapType === "full") {
                            if (type === 'store') {
                                as.getStoreByIdx(idx, function (data) {
                                    apply2storeMap([data]);
                                });
                            } else {
                                as.getEventByIdx(idx, function (data) {
                                    processAllEvents([data]);
                                });
                            }
                        } else {
                            if (type === 'store') {
                                sessionStorage.setItem("store_idx", idx);
                                location.href = "/store/map";  // 주소에 파라미터 안 붙음
                            } else {
                                sessionStorage.setItem("event_idx", idx);
                                location.href = "/store/map";  // 주소에 파라미터 안 붙음
                            }
                        }
                    }
                    // 자동완성 검색어를 선택하지 않을때
                    else if (selectedIndex == -1 && items.length >= 0) {
                        if (!keywordInput.value) {
                            alert("검색어를 입력하시오!");
                            break;
                        }
                        // resetAutocomplete();
                        if (mapType === "full") {
                            mapSearchService(keywordInput.value);
                        } else {
                            sessionStorage.setItem("initialKeyword", keywordInput.value);
                            location.href = "/store/map";  // 주소에 파라미터 안 붙음
                        }
                    }
                    keywordInput.value = "";
                    break;
    
                default:
            }
        });
    
        // 리스트 클릭 이벤트
        autoSearchUL.addEventListener("click", e => {
            deleteAllEle();
    
            const target = e.target.closest("li");
    
            if (!target) return;
    
            const type = target.getAttribute("type");
            const idx = target.getAttribute("idx");

            if (!idx) {
                searchCondition.keyword = '내 주변'
                localStorage.setItem('search_data', JSON.stringify(searchCondition));
                location.href = "/store/map";
            }
    
            // map.jsp
            if (mapType === "full") {
                if (type === "store") {
                    as.getStoreByIdx(idx, function (data) {
                        apply2storeMap([data]);
                    })
                    return;
                } else if (type === "event") {
                    as.getEventByIdx(idx, function (data) {
                        processAllEvents([data]);
                    })
                    return;
                }
            }
            // header
            else {
                if (type === "store") {
                    sessionStorage.setItem("store_idx", idx);
                    location.href = "/store/map";  // 주소에 파라미터 안 붙음
                    return;
                } else if (type === "event") {
                    sessionStorage.setItem("event_idx", idx);
                    location.href = "/store/map";  // 주소에 파라미터 안 붙음
                    return;
                }
            }
    
            const value = target.dataset.value;
            keywordInput.value = value;
            resetAutocomplete();
            mapSearchService(value);
        });
    }

});


// ============================ 마커 관련 함수 ============================

// 지도 마커 추가 (구버전)
function addMarker(map, lat, lng) {
    // 마커가 표시될 위치
    let markerPosition = new kakao.maps.LatLng(lat, lng);
    // 마커 생성
    let marker = new kakao.maps.Marker({
        position: markerPosition
    });
    // 마커가 지도 위에 표시되도록 설정
    marker.setMap(map);
}

/** 마커를 등록하는 함수 */
function registerMarker(vo) {
    let lat, lng, idx, icon;
    const type = vo.type;

    lat = vo[`${type}_lat`];
    lng = vo[`${type}_lng`];
    idx = vo[`${type}_idx`];
    icon = iconObj[`${type}Icon`];
    
    // 마커 위치 설정
    let markerPosition = new kakao.maps.LatLng(lat, lng);
    // 마커 생성
    let marker = new kakao.maps.Marker({
        position: markerPosition,
        // 추후에 마우스 오버시 idx 노출 안되도록 수정
        title: idx,
        image: icon, // 아이콘 이미지 변경 필요
        zIndex: 3
    });
    return marker;
}

// 배열에 추가된 마커들을 지도에 표시하거나 삭제하는 함수
function setMarkers(map, list) {
    for (var i = 0; i < list.length; i++) {
        list[i].marker.setMap(map);
    }
}

/** 배열 비우기 */
function clearMarkers() {
    markerList = [];
}

/** 배열에 추가된 마커를 지도에 표시하는 함수 */
function showMarkers(map, list) {
    setMarkers(map, list)
}

/** 배열에 추가된 마커를 지도에서 삭제하는 함수 */
function hideMarkers(list) {
    setMarkers(null, list);
}

// ============================= 오버레이 관련 함수 =============================
let storeOverlayList = [];
let eventOverlayList = [];

/** 커스텀 오버레이를 등록하는 함수 */
function registerOverlay(vo) {
    const { type } = vo;

    if (!type) return null;

    const idx = vo[`${type}_idx`];
    const lat = vo[`${type}_lat`];
    const lng = vo[`${type}_lng`];

    const nameKey = type === 'store' ? 'store_name' : 'event_title';
    const name = vo[nameKey];

    const content = `
        <div class="customoverlay" id="${type}" idx="${idx}" onclick="clickOverlay(this)">
            <span class="title">${name}</span>
        </div>`;

    return new kakao.maps.CustomOverlay({
        position: new kakao.maps.LatLng(lat, lng),
        content: content,
        yAnchor: 1,
        zIndex: 5,
        clickable: false
    });
}


/** 커스텀 오버레이를 보여주는 함수 */
function showOverlay(map, overlayList) {
    if (basicMap.getLevel() < 4) {
        for (let i = 0; i < overlayList.length; i++) {
            const element = overlayList[i].overlay;
            element.setMap(map);
        }
    }
    preventOverlayClickBlock();
}

/** 커스텀 오버레이를 삭제하는 함수 */
function hideOverlay(overlayList) {
    for (let i = 0; i < overlayList.length; i++) {
        const element = overlayList[i].overlay;
        element.setMap(null);
    }
}

let clickedOverlay = null;

/** 오버레이 클릭 이벤트 추가 함수 */
function clickOverlay(ele) {
    if (clickedOverlay != null) {
        clickedOverlay.setZIndex(5);
    }

    const type = ele.getAttribute("id");
    const idx = ele.getAttribute("idx");
    const li = searchEleByTitle(idx, type);

    viewSideBar = document.querySelector(`.side-bar#${type}`);

    emphMarker(idx, type);
    viewDetailModalPage(li, type);

    showListSideBar();
    showviewSideBar();
    onToggle();
    viewSideBarCheck = true;

    scorllToLi(ele);

    emphOverlay(idx, type);
}

/** 오버레이를 강조하는 함수 */
function emphOverlay(idx, type) {
    const list = VOListObj[`${type}`];
    
    let overlay;

    if (basicMap.getLevel() > 3) {
        hideOverlay(storeVOList);
        hideOverlay(eventVOList);
    }
    list.forEach(vo => {
        if(vo[`${type}_idx`] == idx) {
            overlay = vo.overlay;
            overlay.setMap(basicMap);
            overlay.setZIndex(8);
            clickedOverlay = overlay;
        }
    })

    preventOverlayClickBlock();
}

/** 오버레이 DOM 클릭 방지하는 함수 */
function preventOverlayClickBlock() {
    document.querySelectorAll('.customoverlay').forEach(function (overlay) {
        var container = overlay.parentElement;
        if (container && container.style) {
            // z-index는 유지, 클릭만 통과
            container.style.pointerEvents = 'none';
        }

        // 내부 실제 텍스트(.title 등)는 클릭 가능하게 유지
        overlay.style.pointerEvents = 'auto';
    });
}

// ============================= 이벤트 추가 관련 함수 =============================
/** li요소 클릭 이벤트 추가, 마커 매핑 함수 (ele, type) */
function markerMapping(eles, type) {
    if (eles != null) {
        eles.forEach(ele => {

            // idx 추출
            let idx = ele.getAttribute(`data-${type}_idx`);

            // type 분기
            let voList = [];
            if (type === "store") {
                voList = storeVOList;
            } else if (type === "event") {
                voList = eventVOList;
            }

            // (store or event) 클릭 이벤트 추가
            ele.addEventListener('click', e => {
                hideviewSideBar();
                viewSideBar = document.querySelector(`.side-bar#${type}`);
                // 리스트 중에서 idx 찾기
                voList.forEach(vo => {
                    if (vo.marker.getTitle() === idx) {
                        showviewSideBar();
                        emphMarker(idx, type);
                        onToggle();
                        // 오버레이 표시
                        emphOverlay(idx, type);
                    }
                });
                viewSideBarCheck = true;
                scorllToLi(ele);
            });
        });
    }
}

/** <li>요소를 기준으로 스크롤 조작 함수 */
function scorllToLi(ele) {
    // 마커 클릭시 리스트 사이드바의 스크롤 상태 조작        
    const container = document.querySelector(".side-bar#list #body");
    const offset = ele.offsetTop - container.offsetTop;
    
    container.scrollTo({
    top: offset,
    behavior: 'smooth',
    block : 'start'
    });
    
}

/** 타이틀로 idx가 일치하는 리스트의 요소 찾는 동적 함수 (list, type) */
function searchEleByTitle(title, type) {
    let result;
    const LIS = document.querySelectorAll(`.${type}-card ul li`);
    LIS.forEach(LI => {
        let idx = LI.getAttribute(`data-${type}_idx`);
        if (idx === title) {
            result = LI;
        }
    });
    return result;
}

/** 마커 클릭 이벤트 추가 함수 */
function addMarkerEvent(marker, type) {
    kakao.maps.event.addListener(marker, 'click', function () {
        // 마커 선택
        selectMarker(marker, type);

        let idx = marker.getTitle();
        emphMarker(idx, type);
        let li = searchEleByTitle(idx, type);
        viewDetailModalPage(li, type);

        showListSideBar();
        hideviewSideBar();
        viewSideBar = document.querySelector(`.side-bar#${type}`);

        if (type === "store") {
            showviewSideBar();
        }
        else if (type === "event") {
            showviewSideBar();
        }
        onToggle();
        scorllToLi(li);

        // 오버레이 강조
        emphOverlay(idx, type);
    });
}

// 마커 강조 효과
function selectMarker(marker, type) {
    let icon;
    let oldIcon;

    // 클릭한 마커의 타입에 따른 아이콘 이미지 설정
    if (type === 'store') {
        icon = storeClickedIcon;
    } else if (type === 'event') {
        icon = eventClickedIcon;
    } else {
        icon = clickedIcon;
    }

    // 이미 선택된 마커의 타입에 따른 아이콘 이미지 설정
    if (selectedMarker.type === 'store') {
        oldIcon = storeIcon;
    }
    else if (selectedMarker.type === 'event') {
        oldIcon = eventIcon;
    }

    // 선택된 마커가 없거나 선택된 마커가 해당 마커가 아닐 시에 실행 
    if (!selectedMarker || selectedMarker !== marker) {
        // 기존에 클릭된 마커의 이미지를 바꿈
        // 기존의 마커가 어떤 타입이었는지 구분
        !!selectedMarker.marker && selectedMarker.marker.setImage(oldIcon);

        marker.setImage(icon);
    }
    selectedMarker = { marker: marker, type: type };
}

/** idx를 받으면 일치하는 마커로 이동 및 강조하는 함수 */
function emphMarker(idx, type) {
    let list = [];
    if (type === "store") {
        list = storeVOList;
    } else if (type === "event") {
        list = eventVOList;
    }

    list.forEach(vo => {
        // idx와 일치시 이동 및 강조
        if (idx === vo.marker.getTitle()) {
            // 마커 기준으로 지도 이동
            panToLatLng(basicMap, vo.marker.getPosition().getLat(), vo.marker.getPosition().getLng());
            // 마커 강조
            selectMarker(vo.marker, vo.type);
        }
    });
}

// ================================= 사이드바 관련 함수 =================================
/** 리스트 사이드바 여는 함수 */
function showListSideBar() {
    listSideBar.classList.add("show");
    semiToggle();
}
/** 리스트 사이드바 닫는 함수 */
function hideListSideBar() {
    listSideBar.classList.remove("show");
}

/** 뷰 사이드바 여는 함수 */
function showviewSideBar() {
    viewSideBar.classList.add("show");
    viewSideBarCheck = true;
}
/** 뷰 사이드바 닫는 함수 */
function hideviewSideBar() {
    viewSideBar.classList.remove("show");
}

/** 숫자입력시 해당 pixel만큼 토글버튼의 위치를 설정하는 함수 */
function setToggle(pixel) {
    toggleBtnBox.style.left = `${pixel}px`;
}
/** 토글 닫기 */
function offToggle() {
    toggleBtnBox.classList.remove("semi");
    toggleBtnBox.classList.remove("on");
    toggleBtn.innerHTML = "&gt";
}
/** 토글 중간 */
function semiToggle() {
    toggleBtnBox.classList.add("semi");
    toggleBtnBox.classList.remove("on");
    toggleBtn.innerHTML = "&lt";
}
/** 토글 열기 */
function onToggle() {
    toggleBtnBox.classList.remove("semi");
    toggleBtnBox.classList.add("on");
    toggleBtn.innerHTML = "&lt";
}

/** 리스트 사이드바 토글 함수 */
function toggleListSideBar() {
    // 토글 OFF
    if (listSideBar.classList.contains("show")) {
        hideviewSideBar();
        hideListSideBar();
        offToggle();
    }
    // 토글 ON
    else {
        showListSideBar();
        if (viewSideBarCheck) {
            showviewSideBar();
        }
        if (!viewSideBarCheck) {
            semiToggle();
        } else {
            onToggle();
        }
    }
}

/** 토글 위치 계산하는 함수, 이걸 만들까 말까 만들까 말까 */
function calcToggle() {
}

// 비동기 서비스
const asyncService = (function () {
    function getEventByIdx(idx, callback) {
        fetch(`/modal/event/getByIdx.json`, {
            method: 'POST',
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(idx)
        })
            .then(response => response.json())
            .then(data => {
                if (data != null) {
                    searchCondition.event_idx = data.event_idx;
                    searchCondition.store_idx = null;
                }
                callback(data);
            })
            .catch(err => {
                console.error(err);
            });
    }

    /** idx로 점포 정보 로드하는 함수 */
    function getStoreByIdx(idx, callback) {
        fetch(`/modal/store/getByIdx.json`, {
            method: 'POST',
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(idx)
        })
            .then(response => response.json())
            .then(data => {
                if (data != null) {
                    searchCondition.event_idx = null;
                    searchCondition.store_idx = data.store_idx;
                }
                callback(data);
            })
            .catch(err => {
                console.error(err);
            });
    }

    /** 현위치에서 가장 가까운 점포 n개 로드하는 함수 */
    function getListNearest(condition, amount, callback) {
        condition.amount = amount;
        fetch(`/modal/list/nearest.json`, {
            method: 'POST',
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(condition)
        })
            .then(response => response.json())
            .then(data => {
                callback(data);
            })
            .catch(err => {
                console.error(err);
            });
    }

    /** 반경 내 점포 리스트 (condition, kilometer, callback) */
    function getStoreListWhithin(searchCondition, kilometer, callback) {
        searchCondition.kilometer = kilometer;
        fetch(`/modal/list/within.json`, {
            method: 'POST',
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(searchCondition)
        })
            .then(response => response.json())
            .then(data => {
                callback(data);
            })
            .catch(err => {
                console.error(err);
            });
    }

    // 지역명 기반 검색, 추후에 store_regcode로 진행   
    function getListByReg(store_area, callback) {
        fetch(`/modal/list/store_area/${store_area}.json`)
            .then(response => response.json())
            .then(data => {
                callback(data);
            })
            .catch(err => (err));
    }

    // 현위치 기반 검색
    function getListByLoc(searchCondition, callback) {
        fetch(`/modal/list/loc.json`, {
            method: 'POST',
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(searchCondition)
        })
            .then(response => response.json())
            .then(data => {
                callback(data);
            })
            .catch(err => console.error(err));
    }

    // 카테고리 기반 검색 (store table에 카테고리가 없으니 menu에서 찾아야 하는가)
    function getListByCategory(menu_name, callback) {

    }

    // 점포명 기반 검색
    function getListByName(store_name, callback) {

    }

    /** 키워드 기반 검색 함수 */
    function getListByKeyword(searchCondition, callback) {
        fetch(`/modal/list/keyword.json`, {
            method: 'POST',
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(searchCondition)
        })
            .then(response => response.json())
            .then(data => {
                callback(data);
            })
            .catch(err => {
                console.error(err);
            });
    }

    /** 지역명=키워드 기반 검색 함수 */
    function getListByAddrKeyword(searchCondition, callback) {
        fetch(`/modal/list/addrKeyword.json`, {
            method: 'POST',
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(searchCondition)
        })
            .then(response => response.json())
            .then(data => {
                callback(data);
            })
            .catch(err => {
                console.error(err);
            });
    }

    /** store_idx 입력 시 점포 상세보기 정보 출력 함수 */
    function getStore(store_idx, callback) {
        fetch(`/modal/${store_idx}.json`)
            .then(response => response.json())
            .then(data => {
                callback(data);
            })
            .catch(err => (err));
    }

    /** 메뉴 목록 함수 */
    function getMenuList(store_idx, callback) {
        fetch(`/store/list/${store_idx}.json`)
            .then(response => response.json())
            .then(data => {
                callback(data);
            })
            .catch(err => (err));
    }

    /** 키워드를 입력하여 이벤트 리스트 불러오는 함수 */
    function eventListByKeyword(searchCondition, callback) {
        fetch("/event/eventFilter/keyword", {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(searchCondition)
        })
            .then(response => response.json())
            .then(data => callback(data))
            .catch(err => (err))
    }

    // 함수 객체 리턴
    return {
        getListNearest: getListNearest,
        getStoreListWhithin: getStoreListWhithin,
        getListByReg: getListByReg,
        getListByLoc: getListByLoc,
        getStore: getStore,
        getMenuList: getMenuList,
        getListByKeyword: getListByKeyword,
        getListByAddrKeyword: getListByAddrKeyword,
        eventListByKeyword: eventListByKeyword,
        getStoreByIdx: getStoreByIdx,
        getEventByIdx: getEventByIdx
    };
})();
const as = asyncService;

/** 지도, 위도, 경도를 입력하면 지도 이동 */
function panToLatLng(map, lat, lng) {
    // 대한민국 기준 위도 및 경도 범위
    const isValidLat = lat >= 33.0 && lat <= 43.0;
    const isValidLng = lng >= 124.0 && lng <= 132.0;

    if (isValidLat && isValidLng) {
        const moveLatLng = new kakao.maps.LatLng(lat, lng);
        map.panTo(moveLatLng);
    } else {
        ("대한민국 범위를 벗어난 위도/경도 값입니다:", lat, lng);
    }
}

/** 현재 지도 중심이 주어진 위/경도와 같은지 확인 */
function isSamePosition(map, lat, lng) {
    const center = map.getCenter();
    return (
        Math.abs(center.getLat() - lat) < 0.0001 &&
        Math.abs(center.getLng() - lng) < 0.0001
    );
}

/** 현위치로 이동하는 함수 및  */
function handleCurrentLocationClick(map, lat, lng, mapType) {
    // 이미 현위치라면: 이동하지 않고 커스텀 위치 함수 실행
    if (isSamePosition(map, lat, lng) && mapType === "full") {
        setMyCurrentPlace();
        return;
    }

    // 위치가 다르면 이동
    map.setLevel(3);  // 줌레벨 설정
    const moveLatLng = new kakao.maps.LatLng(lat, lng);
    map.panTo(moveLatLng);

    // 추가 로직: store-loc 일 경우 마커 및 주소 업데이트
    if (mapType === "store-loc") {
        const f = document.querySelector("#store-modify");
        clickMarker.setPosition(moveLatLng);
        initRCodeFromCoords(moveLatLng, f);
        initDetailAddrFromCoords(moveLatLng, f);
    }
}

/** 업로드할 요소 위치를 입력하여 지도를 업로드 함수 (반환값 : 해당 지도 요소) */
function uploadMap(path, lat, lng) {
    let basicOption = { center: new kakao.maps.LatLng(lat, lng), level: 3 };
    let result = new kakao.maps.Map(path, basicOption);
    return result;
}

/** 경도위도를 리턴하는 함수 */
function getLatLng(latlng) {
    (latlng);
}

/** 두 지점 간 간격 구하는 공식 */
function getDistanceFromLatLonInKm(lat1, lon1, lat2, lon2) {
    const R = 6371; // 지구 반지름 (단위: km)
    const dLat = (lat2 - lat1) * Math.PI / 180;
    const dLon = (lon2 - lon1) * Math.PI / 180;
    const a =
        Math.sin(dLat / 2) * Math.sin(dLat / 2) +
        Math.cos(lat1 * Math.PI / 180) *
        Math.cos(lat2 * Math.PI / 180) *
        Math.sin(dLon / 2) * Math.sin(dLon / 2);

    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    const distance = R * c;

    return distance;
}

/** 마커 사이의 거리를 구하는 공식 */
function getDistanceMarkers(marker1, marker2) {
    let latlng1 = marker1.getPosition();
    let latlng2 = marker2.getPosition();
    return getDistanceFromLatLonInKm(latlng1.getLat(), latlng1.getLng(), latlng2.getLat(), latlng2.getLng());
}

// 검색어 정규식
/** "역"으로 끝나는 문자 */
const subwayRegex = /역$/;

/** 검색어 조건 저장 함수 */
function setSubKeyword() {
    localStorage.setItem("search_data", JSON.stringify(searchCondition));
}

/** 지도 검색 기능 서비스 함수 */
function mapSearchService(keyword) {
    showListSideBar();
    hideviewSideBar();
    viewSideBarCheck = false;
    semiToggle();

    // resetAutocomplete();
    // if (!keyword) {
    //     alert("키워드를 입력하시오");
    //     return;
    // }

    storeFailModal.style.display = "none";
    eventFailModal.style.display = "none";

    deleteAllEle();

    searchCondition = {
        level: basicMap.getLevel(),
        lat: basicMap.getCenter().getLat(),
        lng: basicMap.getCenter().getLng(),
        code: centerLoc.code,
        keyword: keyword,
        amount: 100,
        kilometer: setRadiusByLevel(basicMap.getLevel())
    }

    if (keyword === '내 주변') {
        as.getStoreListWhithin(searchCondition, 0.5, (data) => {
            apply2storeMap(data);
            basicMap.setLevel(getMapLevelFromMarkerLists(storeVOList));
            return;
        })
    }

    // 점포 및 장소 검색 시작
    as.getListByAddrKeyword(searchCondition, function (data) {
        if (data.length != 0) {
            apply2storeMap(data);
            let level = getMapLevelFromMarkerLists(storeVOList);
            basicMap.setLevel(level);
        } else {
            searchPlaces(keyword);
        }
    });

    // 이벤트 검색 시작
    as.eventListByKeyword(searchCondition, function (data) {
        if (data.length != 0) {
            processAllEvents(data);
        }
    });
}

// ======= 키워드 검색 관련 =======
/** 장소 검색을 요청하는 함수 */
function searchPlaces(keyword) {
    let ps = new kakao.maps.services.Places();

    if (!keyword.replace(/^\s+|\s+$/g, '')) {
        // alert('키워드를 입력해주세요!');
        return false;
    }

    // 장소검색 객체를 통해 키워드로 장소검색을 요청
    ps.keywordSearch(keyword, placesSearchCB);
}

/** 역 검색을 요청하는 함수 */
function searchSubway(keyword) {
    let ps = new kakao.maps.services.Places();

    if (!keyword.replace(/^\s+|\s+$/g, '')) {
        // alert('키워드를 입력해주세요!');
        return false;
    }

    // 장소검색 객체를 통해 키워드로 장소검색을 요청
    let options = { category_group_code: 'SW8' };
    ps.keywordSearch(keyword, subwaySearchCB, options);
}

/** 역 검색 서비스를 요청하는 함수 */
function serviceSubway(keyword) {
    let ps = new kakao.maps.services.Places();

    if (!keyword.replace(/^\s+|\s+$/g, '')) {
        // alert('키워드를 입력해주세요!');
        return false;
    }

    // 장소검색 객체를 통해 키워드로 장소검색을 요청
    let options = { category_group_code: 'SW8' };
    ps.keywordSearch(keyword, subwayServiceCB, options);
}

// ==================== 장소 검색 서비스 관련 함수 ====================

/** true = 검색 기능을 재실행 하는 여부 */
let againSearch;

/** 장소검색이 완료됐을 때 호출되는 콜백함수 */
function placesSearchCB(data, status, pagination) {
    if (status === kakao.maps.services.Status.OK) {
        placeList = data;
        // (placeList);
        placeInfo = placeList[0];
        if (placeInfo.category_group_name === "지하철역") {
            // ('첫번째 검색어가 지하철입니다.');
            // 지하철 서비스 실행
            serviceSubway(searchCondition.keyword);
        } else {
            as.getListByKeyword(searchCondition, function (data) {
                if (data.length != 0) {
                    apply2storeMap(data);
                }
                else if (placeInfo.category_group_name != "음식점") {
                    // ('일반 음식점은 검색 지원하지 않음');
                    // panToLatLng(basicMap, placeInfo.y, placeInfo.x);
                    // basicMap.setLevel(3);
                    apply2storeMap(data);
                } else {
                    apply2storeMap(data);
                }
            })
        }

    } else if (status === kakao.maps.services.Status.ZERO_RESULT) {
        as.getListByKeyword(searchCondition, function (data) {
            if (data.length != 0) {
                apply2storeMap(data);
            }
            else if (placeInfo.category_group_name != "음식점") {
                // ('일반 음식점은 검색 지원하지 않음');
                // panToLatLng(basicMap, placeInfo.y, placeInfo.x);
                // basicMap.setLevel(3);
                apply2storeMap(data);
            } else {
                apply2storeMap(data);
            }
        });
        return;

    } else if (status === kakao.maps.services.Status.ERROR) {
        apply2storeMap(data);
        // alert('검색 결과 중 오류가 발생했습니다.');
        return;

    }
}

/** 역(subway) 검색이 완료됐을 때 호출되는 콜백함수 */
function subwaySearchCB(data, status, pagination) {
    if (status === kakao.maps.services.Status.OK) {
        subwayInfo = data[0];
    }
    else if (status === kakao.maps.services.Status.ZERO_RESULT) {
        apply2storeMap(data);
        // alert('subway 검색 결과가 존재하지 않습니다.');
        return;
    }
    else if (status === kakao.maps.services.Status.ERROR) {
        apply2storeMap(data);
        // alert('subway 검색 결과 중 오류가 발생했습니다.');
        return;
    }
}

/** 역(subway) 검색이 완료됐을 때 호출되는 콜백함수 (지하철 주변 점포 찾기 서비스를 바로 실행) */
function subwayServiceCB(data, status, pagination) {
    if (status === kakao.maps.services.Status.OK) {
        subwayInfo = data[0];
        // 역으로 화면 이동
        basicMap.setLevel(3);
        panToLatLng(basicMap, subwayInfo.y, subwayInfo.x);
        // 좌표 설정
        searchCondition.lat = subwayInfo.y;
        searchCondition.lng = subwayInfo.x;
        // 역 근처 점포 출력
        as.getStoreListWhithin(searchCondition, 0.5, function (data) {
            apply2storeMap(data);
        });
    }
    else if (status === kakao.maps.services.Status.ZERO_RESULT) {
        apply2storeMap(data);
        // alert('subway 검색 결과가 존재하지 않습니다.');
        return;
    }
    else if (status === kakao.maps.services.Status.ERROR) {
        apply2storeMap(data);
        // alert('subway 검색 결과 중 오류가 발생했습니다.');
        return;
    }
}

/** 검색결과를 지도에 적용하는 콜백함수 (점포) */
function apply2storeMap(data) {
    resetAutocomplete();

    storeUL = document.querySelector(".store-card ul"); // 추후 수정 (시점 문제로 인해 잠시 임시로 재선언)

    if (data.length == 0) {
        failSearch();
        return;
    } else if (data.length == 1) {
        basicMap.setLevel(1);
    }

    // 데이터 등록
    data.forEach(vo => {
        vo.type = "store";

        let marker = registerMarker(vo);
        addMarkerEvent(marker, "store");
        vo.marker = marker;
        
        let overlay = registerOverlay(vo);
        storeOverlayList.push(overlay);
        vo.overlay = overlay;
        
        storeVOList.push(vo);
    });
    (storeVOList);
    
    // 정렬
    sortVOListByDistance(searchCondition.lat, searchCondition.lng, storeVOList);

    // 객체 업데이트
    VOListObj.store = storeVOList;

    let msg = "";
    // 점포 리스트 출력
    storeVOList.forEach(vo => {
        let imgURL = ``;
        if (vo.attach != null) {
            imgURL = `<img src="https://res.cloudinary.com/dbdkdnohv/image/upload/v1747123330/${vo.store_image}_${vo.attach.filename}" alt="${vo.attach.filename}"/>`;
        } else {
            imgURL = `<img src="https://res.cloudinary.com/dbdkdnohv/image/upload/v1747123330/NoImage_pdlhxd.jpg" alt="${vo.store_image}"/>`;
        }
        // (vo);
        msg +=
            // `<li data-store_idx="${vo.store_idx}" name="store_idx">
            `<li data-store_idx="${vo.store_idx}" onclick="viewDetailModalPage(this, 'store')" name ="store_idx">
            ${imgURL}
            <input type="hidden" name="store_address" value="${vo.store_address}">
            <input type="hidden" name="store_activity_time" value="${vo.store_activity_time}">
            <input type="hidden" name="store_num" value="${vo.store_num}">
            <div class="store-description">
                <div class="store-name">${vo.store_name}</div>
                <div class="store-content">${vo.store_content}</div>
            </div>
            <br>
        </li>`;
        storeUL.innerHTML = msg;
    });

    let storeLIS = document.querySelectorAll(".store-card ul li");
    // 가게와 마커를 메핑
    markerMapping(storeLIS, "store");
    // overlayMapping(eventLIS, "store");

    // 오버레이 클릭 이벤트 추가
    let overlayEles = document.querySelector(".customoverlay#store");
    // (overlayEles);

    // overlayEles.forEach(ele => {
    //     (ele);
    //     ele.addEventListener('click', e => {
    //         ('테스트');
    //     })
    // })

    // 마커와 오버레이 표시
    if ((storeMapMode || unitedMapMode) && storeVOList.length != 0) {

        storeListModal.style.display = 'block';
        showMarkers(basicMap, storeVOList);
        showOverlay(basicMap, storeVOList);

        // 스토어 맵일 경우 지도 크기 조절
        // let level = getMapLevelFromMarkerLists(storeVOList);
        // basicMap.setLevel(level);
        panToLatLng(basicMap, storeVOList[0].store_lat, storeVOList[0].store_lng);
        completeSearch();
    }

    setSubKeyword();
}

/** 검색결과를 지도에 적용하는 콜백함수 (이벤트)  */
function apply2eventMap(data) {
    resetAutocomplete();

    let eventUL = document.querySelector(".event-card ul"); // 추후 수정 (시점 문제로 인해 잠시 임시로 재선언)
    // eventUL.innerHTML = "";

    if (data.length == 0) {
        failSearch();
        return;
    }

    let msg = "";
    // 점포 리스트 출력

    data.forEach(vo => {
        let fileList = [];

        // 파일 지정 방식
        // attachFile에서 파일 찾기
        if (vo.attachFile != null) {
            vo.attachFile.forEach(file => {
                if (file.uuid != null) {
                    let imgSrc = `https://res.cloudinary.com/dbdkdnohv/image/upload/v1747269979/${file.uuid}_${file.filename}`;
                    fileList.push(imgSrc);
                } else {
                    let imgSrc = file.filename;
                    fileList.push(imgSrc);
                }
            });
        }
        // cloudinaryFile & externalURL
        else if (vo.cloudinaryFiles != null && vo.externalUrls != null) {
            if (vo.cloudinaryFiles.length != 0) {
                vo.cloudinaryFiles.forEach(file => {
                    let imgSrc = `https://res.cloudinary.com/dbdkdnohv/image/upload/v1747269979/${file.uuid}_${file.filename}`;
                    fileList.push(imgSrc);
                });
            } else if (vo.externalUrls.length != 0) {
                fileList = vo.externalUrls;
            }
        }

        // 이미지  없을 경우
        if (fileList.length == 0) {
            fileList.push("https://res.cloudinary.com/dbdkdnohv/image/upload/v1747123330/NoImage_pdlhxd.jpg");
        }

        msg +=
            // `<li data-store_idx="${vo.store_idx}" name="store_idx">
            `<li data-event_idx="${vo.event_idx}" onclick="viewDetailModalPage(this, 'event')" name="event_idx">
            <img src="${fileList[0]}" alt="event_${vo.event_idx}">
                <div class="event-description">
                    <div class="event-title">${vo.event_title}</div>
                    <div class="event-location">${vo.event_location}</div>
                </div>
            <br>
        </li>`;

        // vo.type = "event";

        let overlay = registerOverlay(vo);
        eventOverlayList.push(overlay);
        vo.overlay = overlay;
    })

    eventUL.innerHTML += msg;

    let eventLIS = document.querySelectorAll(".event-card ul li");
    // 이벤트와 마커를 매핑
    markerMapping(eventLIS, "event");

    // 마커와 오버레이 요소들 표시
    if (eventMapMode || unitedMapMode) {
        eventListModal.style.display = 'block';
        showMarkers(basicMap, eventVOList);
        showOverlay(basicMap, eventVOList);

        // 이벤트 맵일때 크기 조절
        if (eventMapMode) {
            let level = getMapLevelFromMarkerLists(eventVOList);
            basicMap.setLevel(level);
            panToLatLng(basicMap, eventVOList[0].event_lat, eventVOList[0].event_lng);
        }
        completeSearch();
    }


    setSubKeyword();
}

/** 모든 요소를 삭제하는 함수 */
function deleteAllEle() {
    if (mapType != "full") {
        return;
    }

    clickedOverlay = null;

    hideMarkers(storeVOList);
    hideMarkers(eventVOList);
    hideOverlay(storeVOList);
    hideOverlay(eventVOList);

    storeVOList = [];
    eventVOList = [];

    storeOverlayList = [];
    eventOverlayList = [];

    placeList = [];
    placeInfo = {};
    subwayInfo = {};

    document.querySelector(".store-card ul").innerHTML = "";
    document.querySelector(".event-card ul").innerHTML = "";
}

/** 검색결과가 있을 경우 모달을 호출하는 함수 */
function completeSearch() {
    if (storeMapMode && storeVOList.length != 0) {
        storeListModal.style.display = "block"; // 스토어 리스트 view
    }
    else if (eventMapMode && eventVOList.length != 0) {
        eventListModal.style.display = "block";
    } 
    else if (unitedMapMode && (storeVOList.length != 0 || eventVOList.length != 0)) {
        unitedFailModal.style.display = "none";
    }
}

/** 검색결과가 없거나 실패할때 모달을 호출하는 함수 */
function failSearch() {
    // 통합 맵
    if (unitedMapMode) {
        if (storeVOList.length == 0 && eventVOList.length == 0) {
            storeListModal.style.display = "none";
            eventListModal.style.display = "none";
            unitedFailModal.style.display = "block";
        } else if (storeVOList.length != 0 || eventVOList.length != 0) {
            unitedFailModal.style.display = "none";
        }
    }
    // 점포 맵
    else if (storeMapMode) {
        if (storeVOList.length == 0) {
            unitedFailModal.style.display = "block";
        } else {
            unitedFailModal.style.display = "none";
        }
    }
    // 이벤트 맵
    else if (eventMapMode) {
        if (eventVOList.length == 0) {
            unitedFailModal.style.display = "block";
        } else {
            unitedFailModal.style.display = "none";
        }
    }
    viewSideBarCheck = false;
    hideviewSideBar();
    semiToggle();
}

/** 주소를 받으면 좌표를 출력 */
function locationToCoords(loc) {
    return new Promise((resolve, reject) => {
        let geocoder = new kakao.maps.services.Geocoder();
        geocoder.addressSearch(loc, function (result, status) {
    
        if (status === kakao.maps.services.Status.OK) {
            (result);
            let lat = result[0].y;
            let lng = result[0].x;
            (lat);
            (lng);
            (basicMap);
            
            panToLatLng(basicMap, lat, lng);
            clickMarker.setPosition(new kakao.maps.LatLng(lat, lng));
            clickMarker.setMap(basicMap);

            resolve(result); // 변환된 vo를 반환
        }
        else {
            resolve(null);
        }
        });
    })
}

/** 이벤트 좌표 등록 프로미스 */
function address2coordPromise(vo) {
    return new Promise((resolve, reject) => {
        let geocoder = new kakao.maps.services.Geocoder();

        geocoder.addressSearch(vo.event_location, function (result, status) {

            if (status === kakao.maps.services.Status.OK) {
                let lat = result[0].y;
                let lng = result[0].x;
                vo.event_lat = lat;
                vo.event_lng = lng;
                vo.type = "event";

                let marker = registerMarker(vo);

                //("search 테스트");
                // 마커 등록
                vo.marker = marker;

                // 마커에 이벤트 추가
                addMarkerEvent(marker, "event");

                resolve(vo); // 변환된 vo를 반환
            }
            else {
                resolve(null); // 실패한 경우에도 resolve하여 전체 처리를 막지 않음
            }
        });
    });
}

/** 전체 좌표 변환 후 후처리 함수 */
function processAllEvents(data) {
    // (`함수를 ${type} 방식으로 실행`);

    const promises = data.map(vo => address2coordPromise(vo));

    Promise.all(promises).then(results => {
        // null 제거 및 (eventVOList에 추가)
        eventVOList = results.filter(vo => vo !== null);

        // 위치 정렬
        if (mapType === "full") {
            sortVOListByDistance(basicMap.getCenter().getLat(), basicMap.getCenter().getLng(), eventVOList);
        } else {
            sortVOListByDistance(currentLat, currentLng, eventVOList);
        }

        // 키워드 검색
        // ('event vo list : ', eventVOList);
        apply2eventMap(eventVOList);
        VOListObj.event = eventVOList;
        failSearch();

        if (unitedMapMode) {
            if (eventVOList.length > storeVOList.length) {
                let level = getMapLevelFromMarkerLists(eventVOList);
                basicMap.setLevel(level);
                panToLatLng(basicMap, eventVOList[0].event_lat, eventVOList[0].event_lng);
            } else {
                let level = getMapLevelFromMarkerLists(storeVOList);
                basicMap.setLevel(level);
                panToLatLng(basicMap, storeVOList[0].event_lat, storeVOList[0].event_lng);
            }
        }
    });
}

/** 맵의 모드를 변경하는 함수 (문자열로 입력) */
function ctrlMapMode(mode) {
    if (mode === "store") {
        swap2storeMap();
    }
    else if (mode === "event") {
        swap2eventMap();
    }
    else if (mode === "united") {
        swap2unitedMap();
    }
    failSearch();
}

/** 통합 맵으로 변경 */
function swap2unitedMap() {
    document.querySelector("#united-mode").classList.add('select');
    document.querySelector("#event-mode").classList.remove('select');
    document.querySelector("#store-mode").classList.remove('select');
    if (!unitedMapMode) {
        unitedMapMode = true;
        storeMapMode = false;
        eventMapMode = false;

        // 리스트 제어
        if (storeVOList.length != 0) {
            storeListModal.style.display = "block";
        }
        if (eventVOList.length != 0) {
            eventListModal.style.display = "block";
        }

        // 스토어 마커를 맵에 표시
        showMarkers(basicMap, storeVOList);
        // 이벤트 마커를 맵에 표시
        showMarkers(basicMap, eventVOList);

        // 스토어 마커의 오버레이 표시
        showOverlay(basicMap, storeVOList);
        // 이벤트 마커의 오버레이 표시
        showOverlay(basicMap, eventVOList);

        // 상세보기 사이드바 닫기
        hideviewSideBar();
        viewSideBarCheck = false;
        semiToggle();

        // 사이드바를 event로 변경
        // viewSideBar = document.querySelector(".side-bar#united");

        if (storeVOList.length != 0 || eventVOList.length != 0) {
            let level = getMapLevelFromMarkerLists(eventVOList, storeVOList);
            basicMap.setLevel(level);

            if (storeVOList.length > eventVOList.length) {
                panToLatLng(basicMap, storeVOList[0].store_lat, storeVOList[0].store_lng);
            } else {
                panToLatLng(basicMap, eventVOList[0].event_lat, eventVOList[0].event_lng);
            }
        }

        // 검색어 자동완성 설정
        ctrlAutoComplete();
    }
}

/** 이벤트 맵으로 변경 */
function swap2eventMap() {
    document.querySelector("#united-mode").classList.remove('select');
    document.querySelector("#event-mode").classList.add('select');
    document.querySelector("#store-mode").classList.remove('select');
    if (!eventMapMode) {
        eventMapMode = true;
        unitedMapMode = false;
        storeMapMode = false;

        // 화면 삭제
        storeListModal.style.display = "none";
        if (eventVOList.length != 0) {
            eventListModal.style.display = "block";
        }

        // 스토어 마커를 맵에 숨기기
        hideMarkers(storeVOList);
        // 이벤트 마커를 맵에 표시
        showMarkers(basicMap, eventVOList);

        // 스토어 마커의 오버레이 숨기기
        hideOverlay(storeVOList);
        // 이벤트 마커의 오버레이 표시
        showOverlay(basicMap, eventVOList);

        // 상세보기 사이드바 닫기
        hideviewSideBar();
        viewSideBarCheck = false;
        semiToggle();

        if (eventVOList.length != 0) {
            let level = getMapLevelFromMarkerLists(eventVOList);
            basicMap.setLevel(level);
            panToLatLng(basicMap, eventVOList[0].event_lat, eventVOList[0].event_lng);
        }

        // 사이드바를 event로 변경
        viewSideBar = document.querySelector(".side-bar#event");

        // 검색어 자동완성 설정
        ctrlAutoComplete();
    }

}

/** 스토어 맵으로 변경 */
function swap2storeMap() {
    document.querySelector("#united-mode").classList.remove('select');
    document.querySelector("#event-mode").classList.remove('select');
    document.querySelector("#store-mode").classList.add('select');
    if (!storeMapMode) {
        storeMapMode = true;
        eventMapMode = false;
        unitedMapMode = false;

        // 화면 삭제
        eventListModal.style.display = "none";
        if (storeVOList.length != 0) {
            storeListModal.style.display = "block";
        }

        // 이벤트 마커를 맵에 숨기기
        hideMarkers(eventVOList);
        // 스토어 마커를 맵에 표시
        showMarkers(basicMap, storeVOList);

        // 이벤트 마커의 오버레이 숨기기
        hideOverlay(eventVOList);
        // 스토어 마커의 오버레이 표시
        showOverlay(basicMap, storeVOList);

        // 상세보기 사이드바 닫기
        hideviewSideBar();
        viewSideBarCheck = false;
        semiToggle();

        if (storeVOList.length != 0) {
            let level = getMapLevelFromMarkerLists(storeVOList);
            basicMap.setLevel(level);
            panToLatLng(basicMap, storeVOList[0].store_lat, storeVOList[0].store_lng);
        }

        // 사이드바를 store로 재설정
        viewSideBar = document.querySelector(".side-bar#store");

        // 검색어 자동완성 설정
        ctrlAutoComplete();
    }
}

/** 위치정보 커스텀 변경 시작 함수 (map.js와 연동) */
function setMyCurrentPlace() {
    positionOverlay.setMap(basicMap);
    positionOverlay.setPosition(clickMarker.getPosition());

    alert("클릭하여 정확한 위치를 설정하시오");

    customPositionMode = true;
}

/** 위치정보 커스텀 변경 적용 함수 */
function getMyCurrentPlace() {
    customPositionMode = false;

    let latLng = clickMarker.getPosition();
    currentLat = latLng.getLat();
    currentLng = latLng.getLng();

    setPositionData(currentLat, currentLng);
    positionOverlay.setMap(null);
}

/** 위치정보 커스텀 변경 취소 함수 */
function cancelMyCurrentPlace() {
    customPositionMode = false;

    positionOverlay.setMap(null);
    clickMarker.setPosition(new kakao.maps.LatLng(currentLat, currentLng));
}

/** 두 지점 간 거리 계산 함수 (단위: 미터) */
function getDistance(lat1, lng1, lat2, lng2) {
    const R = 6371000; // 지구 반지름 (단위: m)
    const dLat = (lat2 - lat1) * Math.PI / 180;
    const dLng = (lng2 - lng1) * Math.PI / 180;
    const a = Math.pow(Math.sin(dLat / 2), 2) +
        Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
        Math.pow(Math.sin(dLng / 2), 2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    return R * c;
}

// 현위치를 기준으로 List 정렬
function sortVOListByDistance(lat, lng, list) {
    if (list.length == 0) { return; }
    const type = list[0].type; 
    if (type === 'store') {
        list.sort((a, b) => {
            const distA = getDistance(lat, lng, parseFloat(a.store_lat), parseFloat(a.store_lng));
            const distB = getDistance(lat, lng, parseFloat(b.store_lat), parseFloat(b.store_lng));
            return distA - distB;
        });
    }
    else if (type === 'event') {
        list.sort((a, b) => {
            const distA = getDistance(lat, lng, parseFloat(a.event_lat), parseFloat(a.event_lng));
            const distB = getDistance(lat, lng, parseFloat(b.event_lat), parseFloat(b.event_lng));
            return distA - distB;
        });
    }

}

/** 하나 이상의 리스트를 받아 전체 마커 간 최대 거리 확대 레벨 계산 */
function getMapLevelFromMarkerLists(...lists) {
    const allMarkers = [];

    // 모든 리스트에서 .marker가 있는 항목만 수집
    lists.forEach(list => {
        list.forEach(vo => {
            if (vo.marker) {
                allMarkers.push(vo.marker);
            }
        });
    });

    if (allMarkers.length <= 1) return 1;

    let maxDistance = 0;
    for (let i = 0; i < allMarkers.length - 1; i++) {
        const m1 = allMarkers[i].getPosition();
        for (let j = i + 1; j < allMarkers.length; j++) {
            const m2 = allMarkers[j].getPosition();
            const distance = getDistance(m1.getLat(), m1.getLng(), m2.getLat(), m2.getLng());
            if (distance > maxDistance) maxDistance = distance;
        }
    }

    // 거리별 확대 레벨 결정
    if (maxDistance <= 200) return 1;
    if (maxDistance <= 400) return 2;
    if (maxDistance <= 800) return 3;
    if (maxDistance <= 1600) return 4;
    if (maxDistance <= 3200) return 5;
    if (maxDistance <= 6400) return 6;
    if (maxDistance <= 12800) return 7;
    if (maxDistance <= 25600) return 8;
    if (maxDistance <= 51200) return 9;
    if (maxDistance <= 102400) return 10;
    if (maxDistance <= 204800) return 11;
    return 12;
}

/** 맵 줌 상태에 따라 반경을 구하는 함수 */
function setRadiusByLevel(level) {
    let kilometer;
    if (level <= 2) {
        kilometer = 0.5
    }
    else if (level == 3) {
        kilometer = 0.8
    }
    else if (level == 4) {
        kilometer = 1.6
    }
    else if (level == 5) {
        kilometer = 3.2
    }
    else if (level == 6) {
        kilometer = 6.4
    }
    else if (level == 7) {
        kilometer = 12.8
    }
    else if (level == 8) {
        kilometer = 25.6
    }
    else if (level == 9) {
        kilometer = 51.2
    }
    else if (level == 10) {
        kilometer = 102.4
    }
    else if (level == 11) {
        kilometer = 204.8
    }
    else {
        kilometer = 400
    }

    return kilometer;
}

// 자동완성 관련 함수
/** 추천 리스트 UI 갱신 */
function updateSuggestionList(list, type, keyword) {
    // 기존에 있었던 리스트 삭제
    let oldList = document.querySelectorAll(`.${type}-ele`);
    if (oldList != null) {
        oldList.forEach(el => el.remove());
    }

    // autoSearchUL.innerHTML = ""; // 초기화
    if (!keyword) {
        const li = document.createElement("li");
        li.classList.add(`${type}-ele`);
        li.dataset.value = '내 주변';
        li.innerHTML = '내 주변 검색';
        autoSearchUL.appendChild(li);
    }


    list.forEach(item => {
        let text = type === 'event' ? item.event_title : item.store_name;

        const li = document.createElement("li");
        li.classList.add(`${type}-ele`);
        li.setAttribute('type', type);
        li.setAttribute('idx', item[`${type}_idx`]);
        li.dataset.value = text;
        
        // keyword 강조
        if (keyword) {
            const safeKeyword = keyword.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'); // escape
            const regex = new RegExp(`(${safeKeyword})`, "gi");
            const highlighted = 
                text.replace(regex, "<strong>$1</strong>").replace(/\n/g, "").trim();
            li.innerHTML += `<span class="highlighted-text">${highlighted}</span>`;
        } else {
            li.innerHTML += `<span class="highlighted-text">${text}</span>`;
        }

        li.innerHTML += `<span class="ele-type">${type === 'event' ? "이벤트" : "점포"}</span>`;

        autoSearchUL.appendChild(li);
    });

    ctrlAutoComplete();

    if (list.length != 0) {
        autoCompleteBox.style.display = "block";
    }

    selectedIndex = -1;
}


/** 강조 항목 업데이트 */
function updateActiveItem(items) {
    items.forEach((item, i) => {
        item.classList.toggle("active", i === selectedIndex);
        if (i === selectedIndex) {
            // 수정 필요
            item.scrollIntoView({ behavior: "auto", block: "nearest" });
        }
    });
}

/** 자동완성 초기화 */
function resetAutocomplete() {
    hideAutocomplete();
    autoSearchUL.innerHTML = "";
}

/** 자동완성 닫기 */
function hideAutocomplete() {
    autoCompleteBox.style.display = "none";
    selectedIndex = -1;
}

/** 맵 모드에 따라 자동완성 리스트 조절 함수 */
function ctrlAutoComplete() {
    const storeList = document.querySelectorAll(".store-ele");
    const eventList = document.querySelectorAll(".event-ele");

    if (storeMapMode) {
        storeList.forEach(store => {
            store.style.display = 'block';
        });
        eventList.forEach(event => {
            event.style.display = 'none';
        })
    } else if (eventMapMode) {
        eventList.forEach(event => {
            event.style.display = 'block';
        })
        storeList.forEach(store => {
            store.style.display = 'none';
        });
    } else if (unitedMapMode) {
        eventList.forEach(event => {
            event.style.display = 'block';
        })
        storeList.forEach(store => {
            store.style.display = 'block';
        });
    }
}

// 모달 오픈 시 호출되는 함수 내부에서 실행
function initializeEventSlider() {

    const slider = document.querySelector(".event-slider-inner");
    const slides = document.querySelectorAll(".event-slide");
    const prevBtn = document.querySelector(".slider-btn.prev");
    const nextBtn = document.querySelector(".slider-btn.next");

    if (!slider || slides.length === 0) return;

    let currentIndex = 0;
    const totalSlides = slides.length;

    function updateSlidePosition() {
        slider.style.transform = `translateX(-${currentIndex * 100}%)`;
        updateButtonVisibility();
    }

    if (nextBtn) {
        nextBtn.addEventListener("click", () => {
            if (currentIndex < totalSlides - 1) {
                currentIndex++;
                updateSlidePosition();
            }
        });
    }

    if (prevBtn) {
        prevBtn.addEventListener("click", () => {
            if (currentIndex > 0) {
                currentIndex--;
                updateSlidePosition();
            }
        });
    }

    updateSlidePosition(); // 초기 위치 설정
    
    function updateButtonVisibility() {
        prevBtn.classList.toggle("show", currentIndex > 0);
        nextBtn.classList.toggle("show", currentIndex < totalSlides - 1);
    }
}

// ======================== 카카오맵 API 위치<->주소 변환 함수 ========================
/** 경도위도를 입력하면 도로명 주소가 출력되는 함수 */
function searchAddrFromCoords(latlng) {
    let geocoder = new kakao.maps.services.Geocoder();
    let callback = function (result, status) {
        if (status === kakao.maps.services.Status.OK) {
        }
    };
    geocoder.coord2RegionCode(latlng.getLng(), latlng.getLat(), callback);
}

/** 경도위도를 입력하면 주소를 form에 입력하는 함수 */
function initRCodeFromCoords(latlng, form) {
    let geocoder = new kakao.maps.services.Geocoder();
    let callback = function (result, status) {
        if (status === kakao.maps.services.Status.OK) {
            form.regcode.value = result[0].code; // 행정코드
        }
    };
    geocoder.coord2RegionCode(latlng.getLng(), latlng.getLat(), callback);
}

/** 경도위도를 기반으로 현재 지도정보를 최신화 */
function loadAddrFromCoords(latlng) {
    let geocoder = new kakao.maps.services.Geocoder();
    let callback = function (result, status) {
        if (status === kakao.maps.services.Status.OK) {
            centerLoc = result[0];
        }
    };
    geocoder.coord2RegionCode(latlng.getLng(), latlng.getLat(), callback);
}

/** 경도위도로 법정동 상세 주소 정보를 출력하는 함수 */
function initDetailAddrFromCoords(coords, form) {
    let geocoder = new kakao.maps.services.Geocoder();
    let callback = function (result, status) {
        if (status === kakao.maps.services.Status.OK) {
            form.address.value = result[0].address.address_name;
        }
    }
    geocoder.coord2Address(coords.getLng(), coords.getLat(), callback);
}
```

### postcode.js

```java
console.log("postcode load");

let postMap; // 우편번호 지도
document.addEventListener("DOMContentLoaded", () => {

    // 지도 위도 경도 설정 (솔데스크 강남점)
    let latBasic = 37.5054070438773;
    let lngBasic = 127.026682479708;
    // 기본값 설정
    let optionBasic = {center: new kakao.maps.LatLng(latBasic, lngBasic), level: 3};

    let container = document.querySelector('.map#postcode');
    // postMap = new kakao.maps.Map(container, optionBasic);
});

/** 우편번호 서비스 api (map을 입력) */
function pcodeService(map) {
    new daum.Postcode({
        oncomplete: function(data) {
            // 팝업에서 검색결과 항목을 클릭했을때 실행할 코드를 작성하는 부분.

            // 도로명 주소의 노출 규칙에 따라 주소를 표시한다.
            // 내려오는 변수가 값이 없는 경우엔 공백('')값을 가지므로, 이를 참고하여 분기 한다.
            let roadAddr = data.roadAddress; // 도로명 주소 변수
            let extraRoadAddr = ''; // 참고 항목 변수

            // 법정동명이 있을 경우 추가한다. (법정리는 제외)
            // 법정동의 경우 마지막 문자가 "동/로/가"로 끝난다.
            if(data.bname !== '' && /[동|로|가]$/g.test(data.bname)){
                extraRoadAddr += data.bname;
            }
            // 건물명이 있고, 공동주택일 경우 추가한다.
            if(data.buildingName !== '' && data.apartment === 'Y'){
                extraRoadAddr += (extraRoadAddr !== '' ? ', ' + data.buildingName : data.buildingName);
            }
            // 표시할 참고항목이 있을 경우, 괄호까지 추가한 최종 문자열을 만든다.
            if(extraRoadAddr !== ''){
                extraRoadAddr = ' (' + extraRoadAddr + ')';
            }

            // 우편번호와 주소 정보를 해당 필드에 넣는다.
            document.getElementById('postcodeInput').value = data.zonecode;
            document.getElementById("roadAddressInput").value = roadAddr;
            document.getElementById("jibunAddressInput").value = data.jibunAddress;
            
            // 참고항목 문자열이 있을 경우 해당 필드에 넣는다.
            if(roadAddr !== ''){
                document.getElementById("extraAddressInput").value = extraRoadAddr;
            } else {
                document.getElementById("extraAddressInput").value = '';
            }

            let guideTextBox = document.getElementById("guide");
            // 사용자가 '선택 안함'을 클릭한 경우, 예상 주소라는 표시를 해준다.
            if(data.autoRoadAddress) {
                let expRoadAddr = data.autoRoadAddress + extraRoadAddr;
                guideTextBox.innerHTML = '(예상 도로명 주소 : ' + expRoadAddr + ')';
                guideTextBox.style.display = 'block';

            } else if(data.autoJibunAddress) {
                let expJibunAddr = data.autoJibunAddress;
                guideTextBox.innerHTML = '(예상 지번 주소 : ' + expJibunAddr + ')';
                guideTextBox.style.display = 'block';
            } else {
                guideTextBox.innerHTML = '';
                guideTextBox.style.display = 'none';
            }
            // 지도에 마킹
            markByPcode(map, data.address);
        }
    }).open();
}

/** 우편번호 검색 결과를 입력하면 지도에 마킹하는 함수 */
function markByPcode(map, result) {
    clickMarker.setMap(null);

    let resultAddr = result;

    // console.log(resultAddr); // 예시) 경기 성남시 중원구 박석로25번길 44-8
    let geocoder = new daum.maps.services.Geocoder();

    // 주소로 상세 정보를 검색
    geocoder.addressSearch(resultAddr, function(results, status) {

        // 정상적으로 검색이 완료됐으면
        if (status === daum.maps.services.Status.OK) {

            let result = results[0]; //첫번째 결과의 값을 활용
            console.log(result);
            let lat = result.y;
            let lng = result.x;

            panToLatLng(map, lat, lng);
        
            // 해당 주소에 대한 좌표를 받아서
            let coords = new daum.maps.LatLng(result.y, result.x);
            // // 지도를 보여준다.
            // mapContainer.style.display = "block";
            // map.relayout();
            // // 지도 중심을 변경한다.
            // map.setCenter(coords);
            // // 마커를 결과값으로 받은 위치로 옮긴다.
            let marker = new daum.maps.Marker({
                position: new daum.maps.LatLng(37.537187, 127.005476),
                map: map
            });

            marker.setPosition(coords);
        }
    });
}
```

### postcodeForm.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"

    pageEncoding="UTF-8"%>

<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<script src="//t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js"></script>

<script type="text/javascript" src="/resources/js/postcode.js"></script>

<div>

    <div class="map" id="postcode" style="width:500px;height:400px;"></div>

    <div class="form" id="postcode">

        <input type="text" name="event_postcode" id="postcodeInput" placeholder="우편번호" required readonly

            <c:if test="${not empty evo.event_postcode}">

                value="${evo.event_postcode}"

            </c:if>

        >

        <span id="postcodeWarning" style="color: red; font-size: 0.9em; display: none;">우편번호는 필수 입력 항목입니다.</span>

        <button class="mapBtn" id="search-postcode" onclick="">우편번호 찾기</button><br>

  

        <input type="text" name="event_location" id="roadAddressInput" placeholder="도로명주소" readonly

            <c:if test="${not empty evo.event_location}">

                value="${evo.event_location}"

            </c:if>

        >

  

        <input type="text" id="jibunAddressInput" placeholder="지번주소" readonly>

  

        <span id="guide" style="color:#999;display:none"></span>

  

        <input type="text" name="event_location_detail" id="detailAddressInput" placeholder="상세주소"

            <c:if test="${not empty evo.event_location_detail}">

                value="${evo.event_location_detail}"

            </c:if>

        >

  

        <input type="text" id="extraAddressInput" placeholder="참고항목" readonly>

    </div>

</div>
```