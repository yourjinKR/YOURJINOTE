# 개요

프로젝트를 시작 전 사전 미팅에서 협업 툴과 문서 작성 방식에 대해 이야기 나누던 중, 백로그에 대한 얘기가 나왔다.  

백로그가 뭐지?

> **백로그**  
> 제품 백로그는 제품 로드맵 및 그 요구 사항에서 파생된 개발 팀에서 수행할 우선 순위가 지정된 작업 목록입니다. 가장 중요한 항목은 제품 백로그 상단에 표시되므로 팀은 무엇을 가장 먼저 제공해야 하는지 파악할 수 있습니다.


## Github Issue

[Issue와 PR 연결](https://gunu-dev.tistory.com/entry/Git-issue%EC%99%80-pr-%EC%97%B0%EA%B2%B0%ED%95%98%EA%B8%B0feat-%EC%B9%B8%EB%B0%98)

### Github Issue & PR 로 대체하면 안되는걸까?

백로그와 Github Issue &  PR은 엄연히 다르다.  

Issue와 PR은 단지 백로그(단위 task)를 달성하기 위한 수단일 것이다. 




# 실습


![Pasted image 20260305022243](GALLERY/Pasted%20image%2020260305022243.png)

에러 체인 핵심만 보면:

- `BackendApplicationTests > contextLoads() FAILED`
- `BeanCreationException / UnsatisfiedDependencyException`
- 마지막에 `DataSourceProperties$DataSourceBeanCreationException`

즉 DB 커넥션(또는 DB 설정 바인딩)이 안 돼서 DataSource 빈 생성에 실패 → 컨텍스트 로드 실패 → contextLoads 실패 패턴


![Pasted image 20260305023621](GALLERY/Pasted%20image%2020260305023621.png)


# GPT가 설명하는 Github의 기능들

## 기본 (무조건 깔아야 하는 바닥 공사)

- **Organization + Repo 분리 (backend / frontend)**
    
    - 권한(팀/역할), 브랜치 전략, 배포 파이프라인을 분리하기 쉬움.
        
- **Issues**
    
    - 버그/할 일/논의 주제는 “채팅”이 아니라 이슈로 남기기.
        
    - 라벨(backend/frontend/bug/enhancement), 담당자(Assignee), 마일스톤 정도만 써도 효과 큼.
        
- **Pull Request(PR)**
    
    - “main에 직접 push 금지”만 지켜도 품질이 올라감.
        
    - PR 템플릿(체크리스트, 테스트 여부, 스크린샷 등) 같이 쓰면 실수 감소.
        
- **Branch 보호 규칙(Branch protection)**
    
    - main(또는 develop)에 대해:
        
        - PR 필수
            
        - 리뷰 1명 이상 필수
            
        - CI 통과 필수
            
    - 이 3개가 조직 개발의 최소 안전장치.
        
- **README / CONTRIBUTING / CODE_OF_CONDUCT**
    
    - 실행 방법, 환경 변수, 브랜치 규칙, 커밋 규칙을 문서로 고정.
        

## 추천 (팀이 “협업 체감”하는 구간)

- **GitHub Actions (CI)**
    
    - PR 올라오면 자동으로 `build/test/lint` 돌리기.
        
    - “내 컴에서만 됨”을 크게 줄여줌.
        
- **CODEOWNERS**
    
    - 특정 폴더(예: `/backend/**`, `/frontend/**`) 변경 시 자동으로 리뷰어 호출.
        
    - 조직에서 코드 소유권/책임 범위를 깔끔하게 만듦.
        
- **Projects (칸반 보드)**
    
    - 이슈/PR을 보드에 붙여서 진행상태 추적.
        
    - “누가 뭐 하는지” 투명해져서 회의 시간이 줄어듦.
        
- **Issue/PR 템플릿**
    
    - 버그 리포트 양식, 기능 요청 양식, PR 체크리스트를 표준화.
        
- **Releases + Tags**
    
    - v0.1, v0.2 처럼 배포 단위를 버전으로 남기기.
        
    - “어떤 코드가 운영에 나갔는지” 추적이 쉬워짐.
        

## 숙련 (여기부터는 팀 성숙도가 올라간다)

- **환경 분리(Environments) + 배포 승인(Required reviewers)**
    
    - `staging` 자동 배포, `production`은 승인 후 배포 같은 운영 방식.
        
- **Required status checks 세분화**
    
    - “테스트만 통과하면 OK”가 아니라
        
    - lint, typecheck, test, build 등 체크를 분리해서 품질 기준 강화.
        
- **Dependabot**
    
    - 취약점/의존성 업데이트 PR 자동 생성.
        
    - 보안 + 유지보수 비용 절감.
        
- **GitHub Discussions**
    
    - 의사결정/질문/아이디어를 “이슈와 분리”해서 아카이빙.
        
    - 이슈는 “실행 항목”, 디스커션은 “논의”로 역할 분리하면 깔끔.
        
- **Conventional Commits + 자동 릴리즈 노트**
    
    - 커밋 메시지 규칙을 정하면 changelog 생성, 버전 관리 자동화가 쉬움.
        

## 끝판왕 (운영/보안/거버넌스까지 챙기는 레벨)

- **Repository Rulesets (조직 차원의 규칙 일괄 적용)**
    
    - 여러 레포(front/back)에 브랜치 보호/머지 규칙을 한 번에 적용.
        
    - 조직 운영에서 “일관성”이 생김.
        
- **Signed commits / 보호된 태그 / 배포 서명**
    
    - 누가 어떤 코드를 넣었는지 신뢰성을 극대화(공급망 보안).
        
- **Secret scanning + Push protection**
    
    - API 키/토큰 실수로 커밋되는 걸 차단.
        
- **CodeQL (SAST)**
    
    - 정적 분석으로 취약점/위험 패턴을 PR에서 자동 탐지.
        
- **Internal GitHub Packages / Container Registry**
    
    - 공용 라이브러리/도커 이미지를 조직 내부에서 버전 관리.
        
    - 백엔드/프론트가 공통 모듈을 공유할 때 매우 강력.
        
- **자동 배포 파이프라인(Preview Deploy 포함)**
    
    - PR마다 미리보기 배포(프론트는 Preview URL, 백엔드는 Preview API) → 리뷰 품질이 급상승.
        

### 팀프로젝트에 “최소로” 추천 조합(현실적으로 가장 효과 큰 세트)

- PR 기반 개발 + main 브랜치 보호
    
- GitHub Actions로 CI (프론트/백 각각)
    
- CODEOWNERS
    
- Issues + Projects(칸반)
    
- Dependabot(가능하면)
    

원하시면 지금 팀 규모/브랜치 전략(main+feature인지, develop을 둘지)만 알려주시면, **backend/frontend 각각에 맞는 “실제 설정 예시(브랜치 보호 옵션, 폴더 기준 CODEOWNERS, Actions 워크플로우 초안)”**까지 바로 잡아드릴게요.