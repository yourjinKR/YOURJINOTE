
# 실습 기록

### 1차 실패

![[Pasted image 20260305003204.png]]

```
Stderr:
 Container backend-db-1  Recreate
 Container backend-db-1  Error response from daemon: Error when allocating new name: Conflict. The container name "/project-mysql-local" is already in use by container "298bfba8b47f9144d5694f236bb5521f83fe830ede35658d515d28308b5fe18e". You have to remove (or rename) that container to be able to reuse that name.
Error response from daemon: Error when allocating new name: Conflict. The container name "/project-mysql-local" is already in use by container "298bfba8b47f9144d5694f236bb5521f83fe830ede35658d515d28308b5fe18e". You have to remove (or rename) that container to be able to reuse that name.
```

Spring Boot Docker Compose Support는 앱이 실행될 때 `docker-compose.yml`을 읽고 컨테이너를 **새로 생성**(Creating)하려고 시도합니다. 그런데 이미 도커에 `project-mysql-local`이라는 이름의 컨테이너가 (기존에 `docker run`으로 띄웠거나 이전 실행 기록 때문에) 존재하고 있어서 충돌이 발생한 것입니다.

#### 이름 중복 허용

```yaml
services:
  db:
    image: mysql:8.0
    # container_name: project-mysql-local  <-- 이 줄을 삭제하거나 주석 처리
    restart: always
    ...
```

### 로그 파일 

![[Pasted image 20260305004113.png]]

![[Pasted image 20260305004132.png]]

사진과 같이 수많은 파일들이 생성됨

```yml
volumes:
  - ./mysql_data:/var/lib/mysql
```

`docker-compose.yml` 파일의 `volumes` 설정 때문에 아래와 같은 파일이 발생하는 것이다.

> 이 설정은 **컨테이너 내부의 DB 데이터를 내 컴퓨터의 `mysql_data` 폴더에 저장해라**는 뜻입니다. 덕분에 컨테이너를 삭제해도 데이터가 유지되지만, 대신 MySQL이 동작하면서 생성하는 로그 및 임시 파일들이 프로젝트 폴더 안에 무수히 생기게 된 것입니다.

아래와 같이 `gitignore`에 추가하는 것을 권장

```
# MySQL data
mysql_data/
```

https://gemini.google.com/share/b7c509f75e5b