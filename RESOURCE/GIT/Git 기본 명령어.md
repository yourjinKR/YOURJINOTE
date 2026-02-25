# Git 기본 명령어 정리

## 1. 저장소 설정 및 초기화
* `git init`: 새로운 로컬 저장소 생성
* `git clone [url]`: 원격 저장소를 로컬로 복제

## 2. 변경 사항 관리 (Staging & Commit)
* `git status`: 현재 상태 확인 (변경 파일 등)
* `git add [file]`: 파일을 스테이징 영역에 추가 (`.` 사용 시 모든 변경 사항 추가)
* `git commit -m "[message]"`: 스테이징된 변경 사항을 메시지와 함께 커밋

## 3. 원격 저장소 동기화
* `git remote add origin [url]`: 원격 저장소 연결
* `git push origin [branch]`: 로컬 커밋을 원격 저장소로 업로드
* `git pull`: 원격 저장소의 변경 사항을 가져와 로컬에 병합
* `git fetch`: 원격 저장소의 변경 사항을 확인 (병합은 하지 않음)

## 4. 브랜치 관리
* `git branch`: 로컬 저장소의 브랜치 목록 확인
* `git branch -r`: 원격 저장소의 브랜치 목록 확인
* `git branch -a`: 로컬/원격의 모든 브랜치 목록 확인
* `git branch [branch-name]`: 새로운 브랜치 생성
* `git branch -d [branch-name]`: 병합된 브랜치 삭제 (`-D`는 강제 삭제)
* `git branch -m [old-name] [new-name]`: 브랜치 이름 변경
* `git checkout [branch-name]`: 해당 브랜치로 전환
* `git checkout -b [branch-name]`: 새로운 브랜치를 생성하고 즉시 이동
* `git switch [branch-name]`: 브랜치 이동 (최신 권장 명령어)
* `git switch -c [branch-name]`: 브랜치 생성 및 이동 (switch 버전)
* `git merge [branch-name]`: 현재 브랜치에 대상 브랜치의 변경 사항을 병합
* `git rebase [branch-name]`: 현재 브랜치의 커밋을 대상 브랜치 끝으로 재배치

## 5. 변경 사항 확인 및 취소
* `git log`: 커밋 히스토리 조회
* `git diff`: 변경 내용의 차이 확인
* `git reset [file]`: 스테이징 취소
* `git restore [file]`: 파일 변경 사항 되돌리기