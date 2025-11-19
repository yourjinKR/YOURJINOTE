# 무료 배포
https://aamoos.tistory.com/756#google_vignette

# 코틀린이 뭐지?
[[[Kotlin] 코틀린이란]]

# 코틀린 몸풀기
코틀린이 난생 처음이기에 몸풀기를 하고자 이와 같이 진행
[깃허브 레포지토리](https://github.com/yourjinKR/woowacourse-open-mission-kotilin)

# 앱을 만들어야 하나?
[안드로이드-코틀린으로-하는-안드로이드-앱-프로그래밍-1](https://velog.io/@jihyeon9975/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EC%BD%94%ED%8B%80%EB%A6%B0%EC%9C%BC%EB%A1%9C-%ED%95%98%EB%8A%94-%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EC%95%B1-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-1-)
[이것이 안드로이드다 : with 코틀린 : 안드로이드 입문의 3가지 장벽, 언어+실전+환경 완벽 대응!](https://www.snlib.go.kr/hor/menu/11085/program/30009/plusSearchResultDetail.do?searchType=SIMPLE&searchMenuCollectionCategory=&searchMenuEBookCategory=&searchCategory=BOOK&searchKey=ALL&searchKey1=&searchKey2=&searchKey3=&searchKey4=&searchKey5=&searchKeyword=%EC%BD%94%ED%8B%80%EB%A6%B0&searchKeyword1=&searchKeyword2=&searchKeyword3=&searchKeyword4=&searchKeyword5=&searchOperator1=&searchOperator2=&searchOperator3=&searchOperator4=&searchOperator5=&searchPublishStartYear=&searchPublishEndYear=&searchLibrary=&searchPbLibrary=&searchSmLibrary=&searchLibraryArr=MH&searchRoom=&searchKdc=&searchSort=SIMILAR&searchOrder=DESC&searchRecordCount=10&searchBookClass=ALL&currentPageNo=1&viewStatus=IMAGE&preSearchKey=ALL&preSearchKeyword=%EC%BD%94%ED%8B%80%EB%A6%B0&reSearchYn=N&recKey=1849620492&bookKey=1849620494&publishFormCode=BO)

# 코틀린 팀 프로젝트를 하게 된 이유
## 나의 챌린지
- [ ] 실제 서비스 출시해보기
- [ ] 새로운 언어 학습
- [ ] 온라인으로 팀 프로젝트 경험
- [ ] 조금 더체계적인 협업 경험

# 코틀린 공부
과제 풀이, 프로젝트 수행 과정 중 의문점을 중심으로 깊게 알아본 내용을 정리
[[RESOURCE/KOTLIN/[Kotlin] 코틀린의 프로퍼티|[Kotlin] 코틀린의 프로퍼티]]

# 코틀린 프로젝트
## 프로젝트 진행시 처음 경험한 것들
- 온라인 기반 프로젝트
- yaml 설정
- branch 기반 협업
- 코틀린 기반 스프링
- Spring Validator
- [[[wordcapsule] JSP 세팅|JSP 개발환경 세팅]]
## 프로젝트 후 회고
[[[wordcapsule] 트러블 슈팅 및 회고]]
# 개인적인 추가 도전
## 리팩토링
### Model 매핑
[[[wordcapsule] Model를 어떻게 전달할까]]
- [[[Kotlin] 리플렉션 (Reflection)]]
- [[[Kotlin] 확장 함수]]



- 코틀린 확장함수
[[[Kotlin] 코루틴]]
배포 도전?
- docker
- 무료 배고

# 느낀점
## 꼬리에 꼬리를 무는 공부
특정 개념을 이해하기 위해서 다른 무언가를 계속 공부하는 과정을 거쳤다.
또한 직접 연관되어 있지 않더라도 해당 개념에 설명된 예시에서 모르는게 있다면
헤딩 개념 또한 알아봐야 하는 순간들을 경험했다.

꼬리에 꼬리는 무는 궁금증으로 인해 순간 *어, 나 뭐하고 있었지?* 라는 생각이 떠오를 때가 있었다.

물론 빠른 시간 내에 기본 문법만 습득했기에 기초지식 부족으로 인한 일이라고 생각한다.
이런 상황에서 본래 하고자 하는 일을 잊지 않고 방향성 있게 공부하기 위해서는 기록이 중요한거 같다.
### 실제로 경험했던 순간...
- **DTO-Model 매핑 유틸 함수를 만들고 싶다! (시작)**
	- [ ] 확장함수는 뭘까?
	- [x] 코틀린에서 reflection 사용법은 뭘까?
		- [x] 의존성 jar로 추가하는 방법
		- [x] 형변환 과정을 안전하게 하는 법은 뭘까?
			- [x] 다양한 filter 함수
		- [ ] 람다는 비효율적이다?
			- [ ] inline function을 사용해라!
				- [x] 고차 함수란?
					- [x] 코틀린의 람다는 어떨까?
						- [x] reduce와 fold의 차이는? 
						- [x] scope function이란?
				- [x] 컴파일 과정이 어떻길래?
					- [ ] 컴파일 과정 직접 보는 법
				- [x] java의 invoke가 뭐지?
				- [ ] 그럼 inline은 무조건적으로 좋은가?
					- [ ] 응답속도 측정하는 법

