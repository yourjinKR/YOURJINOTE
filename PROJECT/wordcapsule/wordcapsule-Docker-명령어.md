### 환경 설정 및 배포
```bash
# 최초 실행 또는 코드 변경 후 전체 재배포
docker-compose up -d --build

# DB 초기화 포함 클린 재시작 (스키마 오류, 데이터 꼬임 해결)
docker-compose down -v && docker-compose up -d --build

# 현재 docker-compose.yml에 정의된 컨테이너들의 상태 확인
docker-compose ps
```

### 로그 및 상태 확인
```bash
# 컨테이너의 실시간 로그 확인
docker-compose logs -f wordcapsule-app

# 최근 50라인 로그 확인
docker-compose logs --tail 50 wordcapsule-app
```

### 환경 종료 및 정리
```bash
# 실행 중인 모든 컨테이너와 네트워크를 종료 (DB 데이터(Volume)는 유지)
docker-compose down

# 미사용 Docker 이미지 모두 삭제 (디스크 공간 확보)
docker image prune -a

# 모든 컨테이너, 네트워크, 볼륨*, 캐시 등 불필요한 Docker 리소스 일괄 삭제 (개발 환경 정리)
docker system prune -a -f
```

### 정지 및 재시작
```bash
docker-compose stop
docker-compose start
```

### 초기화 후 재시작
DB 스키마 정보가 꼬였을시 실행
```bash
# 1. 컨테이너를 내리면서, DB 데이터가 저장된 볼륨(-v)까지 삭제합니다.
docker-compose down -v

# 2. (선택 사항) 혹시 모르니 빌드 캐시 없이 다시 이미지를 만듭니다.
docker-compose build --no-cache

# 3. 다시 실행합니다.
docker-compose up -d
```