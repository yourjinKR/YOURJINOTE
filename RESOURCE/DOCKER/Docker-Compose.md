# 도커 컴포즈
여러 컨테이너를 동시에 실행시켜주는 녀석
# 핵심 명령어
### docker-compose up
```bash
docker-compose up
```
입력시 아래와 같은 순서로 실행합니다. 이미 생성된 경우 해당 단계를 뜁니다.
1. 서비스를 띄울 네트워크 생성
2. 필요한 볼륨 생성(혹은 이미 존재하는 볼륨과 연결)
3. 필요한 이미지 풀(pull)
4. 필요한 이미지 빌드(build)
5. 서비스 실행 (depends_on 옵션 사용시 서비스 의존성 순서대로 실행)
### --build
이미 빌드가 되있더라도 강제로 빌드를 진행
### -d
백그라운드로 실행
### --force-recreate
docker-compose.yml 파일의 변경점이 없더라도 강제로 컨테이너를 재생성한다.
컨테이너가 종료되었다가 다시 생성
### docker-compose down
서비스를 멈추고 삭제합니다. 컨테이너와 네트워크를 삭제합니다.
### --volume
선언된 도커 볼륨도 삭제합니다.
### docker-compose stop, docker-compose start
서비스를 멈추거나 멈춰 있는 서비스를 시작합ㄴ디ㅏ.
### docker-compose ps
현재 환경에서 실행 중인 각 서비스의 상태를 표시합니다.
### docker-compose logs
컨테이너 로그를 확인합니다.
### -f
tail -f와 유사하게 컨테이너 로그를 실시간으로 확인합니다. (follow)
### docker-compose exec
실행 중인 컨테이너에 해당 명령어를 실행합니다.
```bash
docker-compose exec django ./manage.py makemigrations
docker-compose exec db psql postgres postgres
```
### docker-compose run
특정 명령어를 일회성으로 실행하지만 컨테이너를 batch성 작업으로 사용하는 경우에 해당합니다. 이미 기동하고 있는 컨테이너에 명령어를 실행하고자 하면 docker-compose exec을 사용하는 반면에 docker-compose run을 사용할 경우 컨테이너를 기동시키고 특정 명령어를 실행이 완료된 후에 컨테이너를 종료합니다.
```shell
docker-compose exec web echo "hello world" # 1.
docker-compose run web echo "hello world" # 2.
```
1. 이미 실행된 web 컨테이너에서 echo "hello world"를 실행
2. web 컨테이너에서 echo "hello world"를 실행하고 컨테이너 종료

# 출처
https://byungwoo.oopy.io/2758a690-6ec7-42d0-ae41-5ebc75f4e57a
https://seosh817.tistory.com/387#google_vignette

