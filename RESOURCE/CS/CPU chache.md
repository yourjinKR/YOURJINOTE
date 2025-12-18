
# CPU chache

CPU가 자주 사욯하는 데이터를 매우 빠르게 접근할 수 있도록 보관하는 초고속 기억 장치이다.

## 캐시 동작 원리

### 시간 지역성(Temporal Locality)

- 최근에 사용한 데이터는 가깝게 다시 사용됨  
    예: 반복문에서 i 값

### 공간 지역성(Spatial Locality)

- 특정 주소를 사용하면 그 근처 주소도 곧 사용할 확률이 높음  
    예: 배열 순차 접근
    

### 캐시 히트(Cache Hit)

- CPU가 필요한 데이터가 캐시에 존재 → 매우 빠르게 가져옴

### 캐시 미스(Cache Miss)

- 캐시에 없어서 RAM 또는 디스크에서 불러와야 함 → 느림      
	- Cold miss: 처음 접근이라 없음
	- Capacity miss: 공간 부족
	- Conflict miss: 캐시 충돌(매핑 문제)