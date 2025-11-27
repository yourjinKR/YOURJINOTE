## 컬럼 정보 확인
```sql
-- 컬럼 정보 확인  
show columns from users;
```

## DB 삭제
```sql
-- 초기화  
-- 외래 키 제약 잠시 비활성화  
SET FOREIGN_KEY_CHECKS = 0;  
  
-- 테이블들 전부 삭제 (순서는 크게 상관 없음)  
DROP TABLE IF EXISTS  
    quiz_answers,  
    quiz_record,  
    quiz_options,  
    quizzes,  
    quiz_configs,  
    users;  
  
-- 외래 키 제약 다시 활성화  
SET FOREIGN_KEY_CHECKS = 1;
```

## 스키마
```sql
-- 스키마 생성
CREATE DATABASE IF NOT EXISTS wordcapsule
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_0900_ai_ci;

-- 애플리케이션 전용 계정 (로컬 전용)
CREATE USER IF NOT EXISTS 'wordcapsule'@'localhost' IDENTIFIED BY 'zxcv2372';

-- 권한 부여
GRANT ALL PRIVILEGES ON wordcapsule.* TO 'wordcapsule'@'localhost';
FLUSH PRIVILEGES;

-- 2-1. 계정 상태 확인
SELECT user, host, plugin, account_locked, password_expired
FROM mysql.user
WHERE user = 'wordcapsule';

-- 2-2. 비밀번호 재설정 (확실히 덮어쓰기)
ALTER USER 'wordcapsule'@'localhost' IDENTIFIED BY 'wcapsule!2025';

-- 2-3. 호스트 매칭 이슈 예방: 127.0.0.1, % 도 허용
CREATE USER IF NOT EXISTS 'wordcapsule'@'127.0.0.1' IDENTIFIED BY 'wcapsule!2025';
CREATE USER IF NOT EXISTS 'wordcapsule'@'%'         IDENTIFIED BY 'wcapsule!2025';

GRANT ALL PRIVILEGES ON wordcapsule.* TO 'wordcapsule'@'localhost';
GRANT ALL PRIVILEGES ON wordcapsule.* TO 'wordcapsule'@'127.0.0.1';
GRANT ALL PRIVILEGES ON wordcapsule.* TO 'wordcapsule'@'%';
FLUSH PRIVILEGES;
```


### 스키마를 만들었지만 못찾을때
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