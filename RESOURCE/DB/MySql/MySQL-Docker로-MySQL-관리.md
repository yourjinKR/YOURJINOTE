# Docker-Desktop

### Container Pull

도커 데스크톱에서 `RUN` 버튼 클릭 후 컨테이너 설정 값을 입력 후 컨테이너를 실행 할  수 있다.

![Pasted image 20260127203301](../../../GALLERY/Pasted%20image%2020260127203301.png)
### 환경설정

```sql
-- 1) 실습 DB 생성
CREATE DATABASE IF NOT EXISTS project_main_dev
  DEFAULT CHARACTER SET utf8mb4
  DEFAULT COLLATE utf8mb4_general_ci;

-- 2) 실습 계정 생성 (비번은 편한 걸로)
CREATE USER IF NOT EXISTS 'appuser'@'%' IDENTIFIED BY 'appuser';

-- 3) 권한 부여
GRANT ALL PRIVILEGES ON project_main_dev.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
```

## shell

### 루트, 유저 및 비밀번호 한번에 생성성

```sh
docker run -d --name project-mysql-local \
  -p 3312:3306 \
  -e MYSQL_ROOT_PASSWORD=1234 \
  -e MYSQL_DATABASE=project_main_dev \
  -e MYSQL_USER=appuser \
  -e MYSQL_PASSWORD=appuser \
  mysql:8.0
```

### 컨테이너 진입 → mysql 접속

```sh
docker exec -it project-mysql-local bash
```

```sh
mysql -u root -p
```

### 컨테이너 내 mysql로 바로 진입

```sh
docker exec -it project-mysql-local mysql -u root -p
```