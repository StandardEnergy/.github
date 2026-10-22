# AGENTS.md

AI 코딩 에이전트 공통 작업 지침

- 적용 범위: Standard Energy 조직 저장소 전체
- 사용 방법: 각 저장소 루트에 본 파일 복사하여 사용
  - 조직 `.github` 저장소의 AGENTS.md는 다른 저장소에 자동 적용되지 않음
  - 원본: https://github.com/StandardEnergy/.github/blob/main/AGENTS.md
  - 변경 시 원본 먼저 수정 후 각 저장소에 반영
  - Claude Code 사용 시 저장소 루트 `CLAUDE.md`에 `@AGENTS.md` 한 줄 추가
  - 저장소별 추가 지침은 본 파일 내용 아래에 별도 섹션으로 작성

## 용어 정의

- **단순하고 명백한 수정**: 의도 해석 여지 없고 동작 영향 범위 작은 변경
  - 예: 오타, 주석·문서 문구 수정, 명백한 import 누락, 동작 영향 적은 설정값 변경
  - 운영 환경 설정, 보안·권한 관련 설정 변경은 제외
  - 판단 애매한 경우 단순하지 않은 수정으로 간주

## 원격 저장소

- push 전 `/code-review` 스킬로 리뷰 필수 (단순하고 명백한 수정 포함)
  - 지적 사항 반영 후 push
  - 반영하지 않은 지적 사항은 사유와 함께 사용자에게 보고
  - `/code-review` 스킬 사용 불가 환경이면 그 사실을 알리고 직접 변경 사항 자체 검토
- push 전 로컬에서 변경 사항 피드백 단계 필수
  - 변경 요약과 리뷰 결과 제시 후 사용자 확인 받기
  - 예외: 단순하고 명백한 수정은 리뷰 후 사용자 확인 없이 바로 push 가능
  - Copilot 리뷰 대응 수정도 동일하게 적용 (사용자 확인 후 push, 재리뷰 요청은 push 후 자동)

## 커밋, 브랜치

- 커밋 메시지는 conventional commit 형식, 설명은 한국어 개조식
  - 예: `docs: 설치 가이드 환경 변수 항목 보완`
- 브랜치 이름은 `<conventional commit prefix>/<영문 소문자 작업 요약>` 형식
  - 단어 구분은 하이픈(`-`) 사용
  - 예: `feat/user-search`, `fix/login-session-expiry`

## 이슈, PR

### 공통

- 조직 공통 양식: https://github.com/StandardEnergy/.github
  - 이슈·PR 템플릿: 위 저장소의 `.github/` 디렉터리
  - 저장소별 템플릿이 있으면 저장소 템플릿 우선
- 이슈·PR 템플릿 존재 시 반드시 활용
- 제목·설명 모두 한국어, 개조식 작성
  - 불필요한 조사 최대한 제거
- 제목은 직관적·간결하게, 한국어 사용자에게 익숙한 용어 사용
- 설명에서 이슈·PR·커밋 등 참조 시 항상 링크 함께 제공
  - 같은 저장소: `#123` (GitHub 자동 링크)
  - 다른 저장소: `StandardEnergy/<저장소>#123` 또는 전체 URL
  - 사용자에게 보고할 때도 번호만 쓰지 않고 전체 URL 함께 제공
- 이슈와 PR 함께 작성 시 작업 요청자(GitHub CLI 인증 계정)를 assignee로 지정
  - 예: `gh issue create --assignee @me`, `gh pr create --assignee @me`

### 이슈

- 등록 시 issue type 항상 적절히 지정
  - CLI로 생성 시 템플릿의 type이 자동 적용되지 않으므로 직접 지정
  - 예: `gh issue create --type Bug`, `gh issue edit <이슈 번호> --type Feature`

### PR 생성

- PR 생성 전 `/code-review` 스킬로 리뷰 수행 후 지적 사항 반영
  - 이미 push된 브랜치로 PR만 생성하는 경우에도 적용
  - Copilot 리뷰 사용 한도 절약 목적
- 관련 이슈 연결이 원칙
  - 본문에 `Closes #<이슈 번호>` 형식으로 기재 (PR 템플릿 형식)
  - 다른 저장소 이슈는 `Closes StandardEnergy/<저장소>#<이슈 번호>` 형식
  - 기본 브랜치 외 브랜치 대상 PR은 병합 시 이슈 자동 종료 안 됨, 병합 후 직접 종료
  - 별도 지시 있는 경우에만 이슈 없이 PR 단독 생성
- 제목에 conventional commit prefix 사용
  - 예: `feat: 사용자 목록 검색 기능 추가`, `fix: 로그인 세션 만료 오류 수정`
- 단순하고 명백한 수정 외에는 draft 상태로 생성
- 이미 병합된 PR의 후속 작업인 경우 본문에 선행 PR 명시
  - 예: `#123 후속 작업`
- 웹 프론트엔드 변경 시 (환경 허용하는 경우)
  - 로컬에서 변경 화면 스크린샷 캡처
  - 저장소 외부 임시 디렉터리(Claude Code의 경우 scratchpad 경로)에 저장
  - 저장한 파일 경로 목록을 사용자에게 일괄 제공

### 리뷰 대응 (GitHub Copilot)

- Copilot이 리뷰어로 등록된 경우 리뷰 완료 후 자동으로 리뷰 내용 검토
- 리뷰 지적 사항 대응
  - 대응 내용은 해당 리뷰에 댓글로 기록
  - 해결된 항목은 가능하면 resolve 처리
  - 수정 후 재리뷰 자동 요청
    - 재리뷰 요청 전 `/code-review` 스킬로 리뷰 수행 후 지적 사항 반영
- 리뷰 결과 문제 없으면 draft 자동 해소
- 리뷰로 코드가 지속 수정된 경우 전체 변경 사항 반영하여 PR 본문 재정리
- draft 해소 후 추가 변경 발생 시 다시 draft 전환
  - 단순하고 명백한 수정은 draft 전환·재리뷰 생략 가능

리뷰 요청·draft 상태 전환 명령 예시 (GitHub CLI)

```bash
gh pr edit <PR 번호> --add-reviewer "@copilot"   # Copilot 리뷰(재리뷰) 요청
gh pr ready <PR 번호>                            # draft 해소
gh pr ready <PR 번호> --undo                     # draft 재전환
```

## API

- 백엔드 입력값 검증 필수
- RESTful API 원칙 준수
  - 리소스 중심 URI, HTTP 메서드·상태 코드 의미에 맞게 사용
  - 기존 구현 중 원칙에 어긋나는 부분 발견 시 사용자에게 알리고 수정 방안 제안
- API 문서화용 주석 필수
  - 요청·응답 형식, 오류 응답, 인증 요구 여부 포함

## 코드 품질

- 저장소 기존 작성 관례를 무조건 따르지 않기
  - 부적절한 관례 발견 시 대안 제시
- 수정 사항과 무관한 영역은 기존 포맷 유지
  - 포맷 일괄 변경이 필요하면 별도 작업으로 분리 제안
- 어색하거나 문법에 맞지 않는 네이밍 발견 시 대안 제시

## 문서, 주석

- 조직 공통 양식: https://github.com/StandardEnergy/.github
  - README 작성 템플릿: 위 저장소의 `README.md`
- 문서·코드 주석 모두 한국어, 개조식으로 간결하게 작성
- 한국에서 통용되지 않는 용어 사용 지양
  - 마땅한 대체어 없으면 간단한 설명 덧붙이기
- 전문 용어는 어색한 직역 대신 표준화된 용어 사용
  - 공식 문서 한국어판, 국가·업계 표준 용어집 등에서 확인
  - 예: race condition → 경쟁 상태 (경주 조건 X), idempotent → 멱등 (같은 결과를 내는 X)
  - 업계에서 원어 그대로 통용되는 용어는 원어 유지 (예: 커밋, 브랜치, 리팩터링)
- 이탤릭체 사용 지양 (한글 가독성 저하)
- Windows PowerShell과 Linux Bash 명령어가 다른 경우 예제 구분 작성

예제 구분 형식

````markdown
Windows PowerShell

```powershell
Copy-Item .env.example .env
```

Linux Bash

```bash
cp .env.example .env
```
````

## 테스트

- 코드 변경 시 단위 테스트 함께 추가·수정 (가능한 경우)
- CI에 빌드·테스트가 구성된 경우 로컬에서는 변경 기능 관련 테스트만 수행
  - 전체 테스트는 CI 결과로 확인
- 단순하고 명백한 동작에 대한 단위 테스트는 불필요
  - 예: 단순 getter·setter, 상수 정의

## 접근성

- UI 변경 시 접근성 문제 사전 검토
  - 키보드 조작 가능 여부
  - 스크린 리더용 레이블·대체 텍스트
  - 색상 대비
- 문제 발견 시 PR 본문에 `## 접근성 검토` 섹션 추가하여 검토 결과 기재
