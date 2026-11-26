# release-please 설정 샘플

> [release-please](https://github.com/googleapis/release-please)(Conventional Commits 기반 버전·CHANGELOG 자동화 도구) 도입용 조직 공통 설정 샘플

## 목차

- [구성 파일](#구성-파일)
- [동작 흐름](#동작-흐름)
- [적용 방법](#적용-방법)
- [설정 항목](#설정-항목)
- [release-type 선택](#release-type-선택)
- [배포 이미지 태그 연동](#배포-이미지-태그-연동)
- [주의 사항](#주의-사항)
- [프로젝트 README 기재 예시](#프로젝트-readme-기재-예시)

---

## 구성 파일

| 파일 | 저장소 내 위치 | 용도 |
| --- | --- | --- |
| [release-please-config.json](release-please-config.json) | 루트 | 릴리스 유형, 태그 형식, CHANGELOG 분류 |
| [.release-please-manifest.json](.release-please-manifest.json) | 루트 | 현재 릴리스 버전 (릴리스 PR이 자동 갱신) |

- 실행용 GitHub Actions workflow는 별도 작성 ([적용 방법](#적용-방법), [release-please-action](https://github.com/googleapis/release-please-action) 참고)

## 동작 흐름

1. 기본 브랜치(`main`)에 PR 병합 → 릴리스 PR(`chore(main): release vX.Y.Z`) 자동 생성/갱신
   - 변경 내용: 다음 버전, release-type별 버전 파일(`package.json` 등), `CHANGELOG.md`, `.release-please-manifest.json`
   - 이후 병합되는 PR도 같은 릴리스 PR에 누적
2. 릴리스 PR 병합 → 태그(`vX.Y.Z`) 및 GitHub Release 자동 생성
3. (필요 시) 후속 job에서 태그 기준 빌드, Release asset 첨부, 배포

### 버전 증가 기준

- 병합 커밋 제목(squash 병합 시 PR 제목)의 Conventional Commits 접두어 기준
- 한 릴리스에 여러 접두어가 섞이면 가장 큰 변화 적용 (예: `fix` + `feat` → MINOR)

| 접두어 | 버전 증가 | CHANGELOG |
| --- | --- | --- |
| `feat!:`, `fix(scope)!:`, 본문 `BREAKING CHANGE:` | MAJOR | 노출 |
| `feat` | MINOR | 노출 (새 기능) |
| `fix`, `perf`, `revert`, `refactor`, `build` | PATCH | 노출 |
| `docs`, `ci`, `chore`, `test`, `style` | 없음 (단독 병합 시 릴리스 PR 미생성) | 미노출 |

- 버전 직접 지정: 커밋 본문에 `Release-As: X.Y.Z` footer 추가 (CHANGELOG 노출 접두어 커밋에 기재)
  - `docs`·`chore` 등 미노출 접두어 커밋만 있으면 CHANGELOG 비어 릴리스 PR 미생성 → 지정 버전 반영 안 됨
- 문서가 배포물에 포함되는 프로젝트(패키지에 README 포함 등): `docs` 항목의 `hidden` 제거 고려

### 의존성 변경 접두어

- 배포물에 포함되는 의존성(운영 의존성) 변경: `build(deps)` 사용 → CHANGELOG 노출 및 PATCH 릴리스
- 개발 의존성(테스트·lint 도구 등) 변경: `chore(deps-dev)` 사용 → 배포물 영향 없으므로 릴리스 대상 제외
- GitHub Actions 버전 갱신: `ci(deps)` 사용 → 릴리스 대상 제외
- 자동 업데이트 도구 설정
  - Renovate: `config:recommended` 프리셋 사용 시 운영 의존성 `fix(deps)`, 그 외 `chore(deps)` → `packageRules`로 변경 필요
    - 최상위 `semanticCommitType`은 프리셋의 `packageRules`에 덮어써져 효과 없음
    - `packageRules`는 뒤 규칙이 앞 규칙 덮어씀, `matchPackageNames`의 `*` 패턴은 Renovate v38 이상 지원
    - 의존성 유형(`matchDepTypes`) 이름은 패키지 관리자별로 다름 (npm `dependencies`, Composer `require` 등)
  - Dependabot: `commit-message.prefix`(운영), `prefix-development`(개발)로 지정, `include: scope` 시 `(deps)`·`(deps-dev)` 표기
    - 미지정 시 저장소 커밋 이력 형식을 따라 추정 (`chore(deps)`·`build(deps)`·접두어 없음 등) → 의도와 다른 릴리스 포함·누락 방지 위해 명시 지정
    - `groups`로 묶을 때 운영·개발 의존성이 한 PR에 섞이면 접두어는 하나만 붙음 → 한쪽 변경이 릴리스 분류와 어긋남 (개발 의존성이 PATCH 릴리스에 포함되거나 운영 의존성이 누락)
      - 그룹을 `dependency-type`(`production`·`development`)별로 분리 (npm 등 확인, 미지원 생태계는 병합 전 PR 제목 확인)
    - 설정 변경 전에 열린 Dependabot PR은 제목 그대로 유지 → 병합 후 각 PR에 `@dependabot recreate` 댓글로 재생성
  - 조직 Dependabot 묶음 기준 (전 저장소 · 전 생태계 공통)
    - 패치 갱신: 그룹으로 묶어 PR 1개 (운영·개발 분리 가능한 생태계는 분리)
    - 마이너 갱신: 개별 PR (동작 변경 가능성 → 건별 검토 · 되돌리기 쉽게)
    - 주 버전 갱신: 개별 PR, 미룰 패키지는 해당 PR 에 `@dependabot ignore this major version` 댓글로 건별 제외
      - 주 버전 무시(`ignore` 에 `semver-major`) 는 신규 주 버전 · 보안 수정 누락 위험 → DB 이미지 등 수동 검토가 필요한 패키지에만 지정
      - 전체(`dependency-name: "*"`) 무시는 저장소별 사유가 있는 경우만 유지, 사유를 주석으로 기재
    - `open-pull-requests-limit`: 기본값 5 유지 (생태계 항목별 적용, 초과분은 기존 PR 처리 후 생성)
    - 예외: 함께 갱신해야 하는 의존성은 갱신 유형 무관 그룹 유지 (주석에 사유 기재)
      - 패키지끼리 버전 일치 필요 (예: `@tiptap/*`)
      - `directories` 여러 폴더에 같은 이미지 → 그룹 밖 갱신은 폴더별 개별 PR → 함께 바꿔야 하는 이미지(예: 여러 앱의 node 이미지 + `.nvmrc`)는 `patterns` 그룹
      - digest 고정 이미지(`@sha256:`) → digest 만 바뀌는 갱신은 갱신 유형 판단 불가 → 그룹 `update-types` 생략
    - GitHub Actions 는 대부분 주 버전 태그(`@v4`) 참조 → 패치 묶음은 전체 버전 · SHA 고정 action 에만 적용
    - 그룹 이름 변경 시 기존 이름의 묶음 PR 은 재생성 불가 → 설정 병합 후 직접 닫고 다음 실행 대기

Renovate (`renovate.json`, npm 기준)

```json
{
  "extends": ["config:recommended"],
  "packageRules": [
    { "matchPackageNames": ["*"], "semanticCommitType": "chore" },
    { "matchDepTypes": ["devDependencies"], "semanticCommitScope": "deps-dev" },
    { "matchDepTypes": ["dependencies"], "semanticCommitType": "build" },
    { "matchManagers": ["github-actions"], "semanticCommitType": "ci" }
  ]
}
```

Dependabot (`.github/dependabot.yml`, npm 기준)

```yaml
version: 2
updates:
  - package-ecosystem: npm
    directory: /
    schedule:
      interval: weekly
    commit-message:
      prefix: build
      prefix-development: chore
      include: scope
    # 패치 갱신 → 운영·개발 의존성별 PR 1개씩 묶음 (커밋 접두어 구분), 마이너·주 버전 갱신 → 개별 PR
    groups:
      npm-patch:
        dependency-type: production
        update-types: [patch]
      npm-dev-patch:
        dependency-type: development
        update-types: [patch]
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
    commit-message:
      prefix: ci
      include: scope
    # 패치 갱신 → PR 1개로 묶음, 마이너·주 버전 갱신 → 개별 PR
    groups:
      actions-patch:
        update-types: [patch]
```

## 적용 방법

1. 샘플 파일 복사
   - `release-please-config.json`, `.release-please-manifest.json` → 저장소 루트
2. workflow 작성 (`.github/workflows/release-please.yml`)
   - 트리거: 기본 브랜치 `push`
   - 권한: `contents`, `pull-requests`, `issues`(릴리스 PR 라벨 생성) `write`
   - `concurrency` 지정 (`cancel-in-progress: false`) → 연속 병합 시 릴리스 PR 갱신, 태그 생성 순차 실행
   - `googleapis/release-please-action` 입력: `config-file`, `manifest-file`에 위 파일 지정
   - action 버전은 최신 릴리스 확인 후 커밋 SHA로 고정 ([주의 사항](#주의-사항) 참고)
3. `release-please-config.json` 수정
   - `package-name`: 프로젝트 명
   - `release-type`: 프로젝트 유형에 맞게 지정 ([release-type 선택](#release-type-선택) 참고)
4. 기존 이력 유무에 따라 시작 시점 지정
   - 신규 저장소: 매니페스트 `{}` 유지, 첫 릴리스 버전은 `initial-version`(기본 `1.0.0`)
   - 기존 버전이 있는 저장소: 매니페스트에 현재 버전 기재 (예: `{ ".": "1.3.0" }`)
   - 커밋 이력이 많은 저장소: `bootstrap-sha`에 시작 커밋 지정 → 그 이전 커밋은 첫 CHANGELOG에서 제외
5. 저장소 설정 확인
   - `Settings` → `Actions` → `General` → `Workflow permissions` → `Allow GitHub Actions to create and approve pull requests` 활성화
   - `Settings` → `General` → `Pull Requests` → squash 병합만 허용(merge commit·rebase 비활성화), 기본 커밋 메시지 `Pull request title` 권장
     - merge commit·rebase 병합 시 브랜치의 개별 커밋 제목이 그대로 버전 계산·CHANGELOG에 반영
     - CLI 예: `gh api -X PATCH repos/<조직>/<저장소> -F allow_squash_merge=true -F allow_merge_commit=false -F allow_rebase_merge=false -f squash_merge_commit_title=PR_TITLE -f squash_merge_commit_message=COMMIT_MESSAGES`
     - `COMMIT_MESSAGES`: 브랜치 커밋 메시지가 squash 본문에 합쳐짐 → 본문의 `BREAKING CHANGE:`·`Release-As:` 도 버전 계산에 반영되므로 병합 시 본문 확인 (본문 불필요 시 `PR_BODY`·`BLANK`)
   - 의존성 자동 업데이트 도구 커밋 접두어 지정 ([의존성 변경 접두어](#의존성-변경-접두어) 참고)
6. Conventional Commits 형식의 PR 병합 후 릴리스 PR 생성 확인

## 설정 항목

| 항목 | 샘플 값 | 설명 |
| --- | --- | --- |
| `include-component-in-tag` | `false` | 태그 형식 `vX.Y.Z` (기본값 `<컴포넌트>-vX.Y.Z`) |
| `always-update` | `true` | 릴리스 노트 변화 없어도 실행마다 릴리스 PR을 최신 기본 브랜치 기준으로 재생성 (기본값 `false`, [주의 사항](#주의-사항) 참고) |
| `pull-request-title-pattern` | `chore${scope}: release v${version}` | 릴리스 PR 제목, 조직 PR 제목 규칙(Conventional Commits)과 일치 (`${scope}` → 대상 브랜치, 예: `(main)`) |
| `pull-request-header` | `릴리스 PR — ...` | 릴리스 PR 본문 상단 안내 문구 (기본값은 영문) |
| `release-type` | `<프로젝트 유형>` | 버전 갱신 대상 파일 결정 (필수, [release-type 선택](#release-type-선택) 참고) |
| `package-name` | `<프로젝트 명>` | 릴리스 PR, CHANGELOG에 표시되는 이름 |
| `changelog-path` | `CHANGELOG.md` | CHANGELOG 파일 경로 |
| `changelog-sections` | 11개 접두어 | 접두어별 CHANGELOG 제목(한국어), `hidden: true` 시 CHANGELOG 미노출, 해당 접두어만 병합 시 릴리스 PR 미생성 (노출 접두어와 함께 병합 시 릴리스에 포함) |
| `initial-version` | (미지정) | 첫 릴리스 버전, 미지정 시 `1.0.0` (`python` 유형은 무시하고 `0.1.0` 고정 → [release-type 선택](#release-type-선택) 참고) |
| `bootstrap-sha` | (미지정) | 첫 릴리스 PR의 커밋 수집 시작점 |

- 샘플에 없는 접두어(예: `change`) 사용 시 `changelog-sections`에 항목 추가
- JSON 파일 특성상 주석 불가 → 설정 의도는 workflow 주석 또는 프로젝트 README에 기재
- 전체 항목: [config.json 스키마](https://github.com/googleapis/release-please/blob/main/schemas/config.json), [manifest-releaser 문서](https://github.com/googleapis/release-please/blob/main/docs/manifest-releaser.md)

## release-type 선택

- 릴리스 PR에서 갱신할 버전 기록 파일 결정 (버전 계산, CHANGELOG, 태그/Release 생성은 유형 무관 동일)
- 프로젝트 구성에 맞게 지정 → 샘플 값(`<프로젝트 유형>`) 그대로 사용 불가

| release-type | 갱신 대상 파일 | 적합한 프로젝트 |
| --- | --- | --- |
| `node` | `package.json`, `package-lock.json` | Node.js, 프론트엔드 |
| `go` | 없음 (CHANGELOG만 갱신) | Go (버전은 태그 기준 관리) |
| `maven` | `pom.xml` 등 | Maven 기반 Java |
| `python` | `pyproject.toml`, `setup.py`, `setup.cfg` 등 | Python |
| `rust` | `Cargo.toml`, `Cargo.lock` | Rust |
| `helm` | `Chart.yaml` | Helm 차트 |
| `simple` | 루트 `version.txt` (파일 존재 시) | 전용 유형이 없거나 여러 언어가 섞인 프로젝트 |

- 그 외 지원 유형: [release-please 지원 유형 목록](https://github.com/googleapis/release-please#strategy-language-types-supported)
- 선택 기준
  - 단일 언어 + 전용 유형 존재: 해당 유형 사용
  - 전용 유형 없음(Gradle 등) 또는 혼합 구성(서버 + 클라이언트 등): `simple` + `extra-files`
  - `python` 유형은 `initial-version`을 무시하고 첫 릴리스 `0.1.0` 고정 (release-please 17.x 확인) → 첫 버전 `1.0.0`이 필요하면 `simple` + `extra-files`(`pyproject.toml`, `uv.lock`) 사용

### extra-files

- release-type이 처리하지 못하는 버전 파일 추가 지정
- 형식: `json`, `yaml`, `toml`, `xml`(경로 지정), `generic`(주석 표기)

```json
"release-type": "simple",
"extra-files": [
  { "type": "json", "path": "client/package.json", "jsonpath": "$.version" },
  { "type": "generic", "path": "server/build.gradle" }
]
```

- uv 프로젝트: `pyproject.toml`만 갱신하면 `uv.lock`의 자기 패키지 버전 불일치 → `uv lock --check`·`uv sync --locked` 실패 → `uv.lock`도 함께 지정
  - TOML 파서가 값을 `{value, start, end}` 형태로 감싸므로 조건식에 `.value` 필요 (`@.name==` 은 매칭 실패)

```json
{ "type": "toml", "path": "pyproject.toml", "jsonpath": "$.project.version" },
{ "type": "toml", "path": "uv.lock", "jsonpath": "$.package[?(@.name.value=='<패키지 이름>')].version" }
```

- `generic` 유형: 대상 줄에 `x-release-please-version` 주석 표기 필요

```groovy
version = '0.0.1-SNAPSHOT' // x-release-please-version
```

- Go 바이너리 등 소스에 버전을 두지 않는 경우: 빌드 시 태그 주입 (`go build -ldflags "-X main.version=${TAG}"`)

### 릴리스 산출물 첨부

- release-please job 이후 후속 job에서 빌드 단계 작성
- release-please 출력값 활용: `release_created`(릴리스 생성 여부), `tag_name`(`vX.Y.Z`)
  - step 출력은 후속 job에서 직접 참조 불가 → release-please job의 `outputs`로 승격 후 `needs.<job>.outputs.*`로 참조
- Release asset 업로드: `gh release upload "${TAG_NAME}" <파일> --clobber` 또는 `softprops/action-gh-release`
  - `gh` CLI 사용 시 step `env`에 `GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}` 지정 필요 (Actions 토큰 자동 인증 미지원)
  - self-hosted 러너는 `gh` CLI 설치 여부 확인
- 아래 예시의 action 버전은 작성 시점(2026-10) 기준 → 적용 시 각 action 저장소 Releases에서 최신 버전 확인 후 커밋 SHA 갱신

```yaml
jobs:
  release_please:
    runs-on: self-hosted
    outputs:
      release_created: ${{ steps.release.outputs.release_created }}
      tag_name: ${{ steps.release.outputs.tag_name }}
    steps:
      - uses: googleapis/release-please-action@45996ed1f6d02564a971a2fa1b5860e934307cf7 # v5.0.0
        id: release
        with:
          config-file: release-please-config.json
          manifest-file: .release-please-manifest.json

  upload_release_asset:
    needs: release_please
    if: ${{ needs.release_please.outputs.release_created == 'true' }}
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
        with:
          ref: ${{ needs.release_please.outputs.tag_name }}
      # 빌드 단계 생략
      - name: Release asset 업로드
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          TAG_NAME: ${{ needs.release_please.outputs.tag_name }}
        run: gh release upload "${TAG_NAME}" dist/* --clobber
```
- 빌드 실패 대비: `workflow_dispatch` 입력으로 태그를 받아 재빌드/재첨부하는 경로 마련 권장

## 배포 이미지 태그 연동

- 컨테이너 이미지 태그를 릴리스 버전과 일치시켜 배포 이력 · 롤백 기준 통일
- 수동 배포 workflow(`workflow_dispatch`)에서 태그를 직접 조회 → `GITHUB_TOKEN` 태그 push 트리거 제약과 무관
- 적용 예: jarvis, Vector, visionary-ems, vveather, se-ess-lpms-proxy, kakao-mass 의 `.github/workflows/deploy.yml`

| 단계 | 내용 |
| --- | --- |
| 버전 결정 | 입력 `release_version`(X.Y.Z, v 제외), 비우면 최신 릴리스 태그 |
| 코드 | 해당 태그 checkout 후 빌드 · 배포 (`actions/checkout` 에 `fetch-depth: 0` → 태그 조회) |
| 이미지 태그 | 릴리스 버전 그대로 사용 (`RELEASE_VERSION` Secret 등 별도 지정 불필요) |
| 존재 확인 | ghcr 에 같은 버전 이미지 있으면 빌드 · push 생략 → 롤백 시 재빌드 없음, 릴리스 이미지 덮어쓰기 방지 |
| job 간 전달 | 결정 버전을 build job `outputs` 로 deploy job 에 전달 → 두 job 이 같은 태그 사용 |

배포 버전 결정 step (입력값 검증, 최신 정식 릴리스 태그 조회 — rc 등 사전 릴리스 제외)

```yaml
- name: 배포 버전 결정
  id: version
  env:
    INPUT_VERSION: ${{ inputs.release_version }}
  run: |
    set -euo pipefail
    if [ -n "$INPUT_VERSION" ]; then
      [[ $INPUT_VERSION =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || { echo "::error::release_version 형식 오류 — X.Y.Z (v 제외)"; exit 1; }
      tag="v$INPUT_VERSION"
      git rev-parse -q --verify "refs/tags/$tag" > /dev/null || { echo "::error::릴리스 태그 $tag 없음"; exit 1; }
    else
      tag=$(git tag --list 'v*' --sort=-v:refname | grep -E '^v[0-9]+\.[0-9]+\.[0-9]+$' | head -1 || true)
      [ -n "$tag" ] || { echo "::error::릴리스 태그 없음 — 릴리스 PR 병합(태그 생성) 후 배포"; exit 1; }
    fi
    git checkout -q --detach "refs/tags/$tag"
    echo "version=${tag#v}" >> "$GITHUB_OUTPUT"
```

- 입력값은 `env` 로 전달 (`${{ inputs.* }}` 를 `run` 에 직접 넣으면 셸 코드로 해석될 수 있음)
- 이미지 존재 확인: `docker manifest inspect <이미지>` 실패 메시지가 `manifest unknown` · `name unknown` 일 때만 '없음' 판정
  - 네트워크 · 인증 등 다른 오류를 '없음'으로 보면 기존 릴리스 이미지를 다시 빌드해 덮어씀 → 실패 처리
  - ghcr 는 태그 없음 · 패키지 없음 모두 `manifest unknown` 반환
  - 일부 이미지만 있으면(이전 push 중단 등) 없는 이미지만 push
- 배포 workflow 파일은 실행 시 선택한 브랜치(기본 `main`) 기준, compose 파일 등은 태그 기준 → 구조가 크게 바뀐 오래된 태그는 배포 실패 가능
- 릴리스 태그가 생기기 전(첫 릴리스 PR 병합 전)에는 배포 불가 → 도입 직후 릴리스 PR 병합 필요

## 주의 사항

- 연속 병합 시 릴리스 PR에 이전 시점 버전 파일 반영 가능 → `always-update: true` 유지
  - 실행 도중 다음 PR 병합 시 커밋 목록(릴리스 노트)은 최신, 버전 파일(`package.json` 등)은 이전 시점 기준으로 릴리스 커밋 생성
  - `always-update` 미지정 시 다음 실행은 릴리스 노트 동일 → 릴리스 PR 갱신 안 함 (`remained the same`)
  - 증상 예: Dependabot PR 연속 병합 후 릴리스 PR의 의존성 버전 되돌아감 → lockfile 불일치로 CI 실패
- 릴리스 PR 병합 전 마지막 PR의 workflow 실행 완료 확인
  - 실행 완료 전 병합 시 해당 PR이 코드에는 포함되나 CHANGELOG·릴리스 노트에서 누락 (다음 릴리스에도 미표시)
- `GITHUB_TOKEN`으로 생성한 릴리스 PR·태그는 다른 workflow(CI, 태그 push 트리거 배포 등)를 실행하지 않음
  - 필수 상태 검사(branch protection)가 걸린 경우 릴리스 PR 병합 불가 → GitHub App 토큰 또는 PAT를 `token` 입력으로 전달
  - 태그 기반 후속 작업은 같은 workflow 내 후속 job으로 처리 권장
  - 수동 배포는 배포 workflow 에서 태그를 직접 조회하는 방식 가능 ([배포 이미지 태그 연동](#배포-이미지-태그-연동) 참고)
- 커밋 1개인 PR은 squash 병합 시 커밋 메시지가 기본 병합 메시지로 사용 → 커밋 메시지도 Conventional Commits 형식 유지 (기본 커밋 메시지를 PR 제목으로 설정 시 해당 없음)
- action 버전은 태그(`@v5`) 대신 커밋 SHA로 고정 (쓰기 권한 + self-hosted 러너 실행 → 태그 변조 시 영향 큼)
  - 버전 갱신: Dependabot(`package-ecosystem: github-actions`) 활용 권장 (커밋 접두어: [의존성 변경 접두어](#의존성-변경-접두어) 참고)

## 프로젝트 README 기재 예시

```markdown
## 릴리스

- [release-please](https://github.com/googleapis/release-please) 기반 버전 관리 (`.github/workflows/release-please.yml`)
- PR 제목 [Conventional Commits](https://www.conventionalcommits.org/ko/v1.0.0/) 형식 필수 (squash 병합 커밋 = PR 제목)
  - `feat` → MINOR, 그 외 CHANGELOG 노출 접두어 → PATCH, `!` 표기 → MAJOR
  - `docs`, `ci`, `chore`, `test`, `style`: CHANGELOG 제외, 단독으로는 릴리스 PR 미생성
  - 의존성 변경: 운영 `build(deps)`, 개발 `chore(deps-dev)`, GitHub Actions `ci(deps)`
- 릴리스 흐름
  1. `main` 병합 시 릴리스 PR(`chore(main): release vX.Y.Z`) 자동 생성/갱신
  2. 릴리스 PR 병합 시 태그(`vX.Y.Z`) 및 GitHub Release 자동 생성
- 설정: `release-please-config.json`, 현재 버전: `.release-please-manifest.json`
```
