# MySQL schema
[스키마 생성 방법](https://m.blog.naver.com/calb30095/222054766859)
MySQL에서는 **스키마**와 **데이터베이스**가 같은 뜻임
![[Pasted image 20251111013947.png]]
스키마(데이터베이스)안에 테이블이 있고, 그 테이블 안에 데이터가 있다.

## 스키마를 만들었지만 못찾을때
스키마(DB)는 존재하지만, 그 스키마에 붙을 애플리케이션 계정이 없어서 `Access denied`가 난 거예요.
아래 SQL을 Workbench에서 **root로 접속한 세션**에서 그대로 실행해 계정과 권한을 만들어 주세요.
```sql
-- 1) 애플리케이션 전용 계정 생성(세 호스트 모두: localhost, 127.0.0.1, %)
CREATE USER IF NOT EXISTS 'wordcapsule'@'localhost'   IDENTIFIED BY 'zxcv2372';
CREATE USER IF NOT EXISTS 'wordcapsule'@'127.0.0.1'   IDENTIFIED BY 'zxcv2372';
CREATE USER IF NOT EXISTS 'wordcapsule'@'%'           IDENTIFIED BY 'zxcv2372';

-- 2) 스키마 권한 부여
GRANT ALL PRIVILEGES ON wordcapsule.* TO 'wordcapsule'@'localhost';
GRANT ALL PRIVILEGES ON wordcapsule.* TO 'wordcapsule'@'127.0.0.1';
GRANT ALL PRIVILEGES ON wordcapsule.* TO 'wordcapsule'@'%';

FLUSH PRIVILEGES;

-- 3) 생성 확인
SELECT user, host, plugin, account_locked, password_expired
FROM mysql.user
WHERE user = 'wordcapsule';

-- 4) 권한 확인(선택)
SHOW GRANTS FOR 'wordcapsule'@'localhost';
SHOW GRANTS FOR 'wordcapsule'@'127.0.0.1';
SHOW GRANTS FOR 'wordcapsule'@'%';

```

## 스키마 확인
### 1. 클라이언트(WorkBench/DataGrip)에서 스키마 선택/새로고침

### 2. SQL로 바로 확인
- **DataGrip**: 좌측 데이터소스 우클릭 → **Schemas…** → `wordcapsule` 체크 → **Apply** → 좌측 트리 **Refresh(Ctrl+F5)**.
- **MySQL Workbench**: SCHEMAS 패널에서 `wordcapsule` 선택 → 우클릭 **Refresh All**.
```sql
-- 내가 보고 있는 DB가 맞는지
SELECT DATABASE();

-- 스키마 목록
SHOW DATABASES;

-- 스키마 선택
USE wordcapsule;

-- 테이블 목록
SHOW TABLES;

-- 생성된 테이블 구조 확인
DESCRIBE quiz_records;
DESCRIBE quiz_configs;
```

### 3. 연결 대상이 같은 서버/포트인지 점검
앱은 `jdbc:mysql://localhost:3306/wordcapsule`로 붙습니다.  
툴에서도 **Host=localhost, Port=3306** 인지 확인하세요. (가끔 127.0.0.1/3307로 연결되어 다른 인스턴스를 보고 있는 경우가 있습니다.)
```sql
-- 현재 인스턴스 포트 확인
SHOW VARIABLES LIKE 'port';
SELECT @@hostname, @@port;
```

### 4) 권한/호스트 이슈가 있으면 (보기 권한 부족 시)
이미 ‘wordcapsule@localhost’로 생성했더라도, 툴 접속이 `127.0.0.1`이면 별도 계정이 필요합니다.
```sql
-- 필요 시 둘 다 부여
GRANT ALL PRIVILEGES ON wordcapsule.* TO 'wordcapsule'@'localhost';
GRANT ALL PRIVILEGES ON wordcapsule.* TO 'wordcapsule'@'127.0.0.1'; 
FLUSH PRIVILEGES;`
```




```
```