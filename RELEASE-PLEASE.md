# release-please 설정 샘플

> [release-please](https://github.com/googleapis/release-please)(Conventional Commits 기반 버전·CHANGELOG 자동화 도구) 도입용 조직 공통 설정 샘플

## 목차

- [구성 파일](#구성-파일)
- [동작 흐름](#동작-흐름)
- [적용 방법](#적용-방법)
- [설정 항목](#설정-항목)
- [release-type 선택](#release-type-선택)
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
| `fix`, `perf`, `revert`, `refactor`, `build`, `docs` | PATCH | 노출 |
| `ci`, `chore`, `test`, `style` | 없음 (단독 병합 시 릴리스 PR 미생성) | 미노출 |

- 버전 직접 지정: 커밋 본문에 `Release-As: X.Y.Z` footer 추가 (CHANGELOG 노출 접두어 커밋 한정)

## 적용 방법

1. 샘플 파일 복사
   - `release-please-config.json`, `.release-please-manifest.json` → 저장소 루트
2. workflow 작성 (`.github/workflows/release-please.yml`)
   - 트리거: 기본 브랜치 `push`
   - 권한: `contents`, `pull-requests`, `issues`(릴리스 PR 라벨 생성) `write`
   - `concurrency` 지정 (`cancel-in-progress: false`) → 연속 병합 시 릴리스 PR 갱신, 태그 생성 순차 실행
   - `googleapis/release-please-action` 입력: `config-file`, `manifest-file`에 위 파일 지정
   - action 버전은 커밋 SHA로 고정 ([주의 사항](#주의-사항) 참고)
3. `release-please-config.json` 수정
   - `package-name`: 프로젝트 명
   - `release-type`: 프로젝트 유형에 맞게 지정 ([release-type 선택](#release-type-선택) 참고)
4. 기존 이력 유무에 따라 시작 시점 지정
   - 신규 저장소: 매니페스트 `{}` 유지, 첫 릴리스 버전은 `initial-version`(기본 `1.0.0`)
   - 기존 버전이 있는 저장소: 매니페스트에 현재 버전 기재 (예: `{ ".": "1.3.0" }`)
   - 커밋 이력이 많은 저장소: `bootstrap-sha`에 시작 커밋 지정 → 그 이전 커밋은 첫 CHANGELOG에서 제외
5. 저장소 설정 확인
   - `Settings` → `Actions` → `General` → `Workflow permissions` → `Allow GitHub Actions to create and approve pull requests` 활성화
   - `Settings` → `General` → `Pull Requests` → squash 병합 허용, 기본 커밋 메시지 `Pull request title` 권장
6. Conventional Commits 형식의 PR 병합 후 릴리스 PR 생성 확인

## 설정 항목

| 항목 | 샘플 값 | 설명 |
| --- | --- | --- |
| `include-component-in-tag` | `false` | 태그 형식 `vX.Y.Z` (기본값 `<컴포넌트>-vX.Y.Z`) |
| `pull-request-title-pattern` | `chore${scope}: release v${version}` | 릴리스 PR 제목, 조직 PR 제목 규칙(Conventional Commits)과 일치 (`${scope}` → 대상 브랜치, 예: `(main)`) |
| `pull-request-header` | `릴리스 PR — ...` | 릴리스 PR 본문 상단 안내 문구 (기본값은 영문) |
| `release-type` | `<프로젝트 유형>` | 버전 갱신 대상 파일 결정 (필수, [release-type 선택](#release-type-선택) 참고) |
| `package-name` | `<프로젝트 명>` | 릴리스 PR, CHANGELOG에 표시되는 이름 |
| `changelog-path` | `CHANGELOG.md` | CHANGELOG 파일 경로 |
| `changelog-sections` | 11개 접두어 | 접두어별 CHANGELOG 제목(한국어), `hidden: true` 시 미노출 및 버전 증가 대상 제외 |
| `initial-version` | (미지정) | 첫 릴리스 버전, 미지정 시 `1.0.0` |
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

```yaml
jobs:
  release_please:
    runs-on: self-hosted
    outputs:
      release_created: ${{ steps.release.outputs.release_created }}
      tag_name: ${{ steps.release.outputs.tag_name }}
    steps:
      - uses: googleapis/release-please-action@<커밋 SHA> # vX.Y.Z
        id: release
        with:
          config-file: release-please-config.json
          manifest-file: .release-please-manifest.json

  upload_release_asset:
    needs: release_please
    if: ${{ needs.release_please.outputs.release_created == 'true' }}
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
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

## 주의 사항

- 릴리스 PR 병합 전 마지막 PR의 workflow 실행 완료 확인
  - 실행 완료 전 병합 시 해당 PR이 코드에는 포함되나 CHANGELOG·릴리스 노트에서 누락 (다음 릴리스에도 미표시)
- `GITHUB_TOKEN`으로 생성한 릴리스 PR·태그는 다른 workflow(CI, 태그 push 트리거 배포 등)를 실행하지 않음
  - 필수 상태 검사(branch protection)가 걸린 경우 릴리스 PR 병합 불가 → GitHub App 토큰 또는 PAT를 `token` 입력으로 전달
  - 태그 기반 후속 작업은 같은 workflow 내 후속 job으로 처리 권장
- 커밋 1개인 PR은 squash 병합 시 커밋 메시지가 기본 병합 메시지로 사용 → 커밋 메시지도 Conventional Commits 형식 유지 (기본 커밋 메시지를 PR 제목으로 설정 시 해당 없음)
- action 버전은 태그(`@v5`) 대신 커밋 SHA로 고정 (쓰기 권한 + self-hosted 러너 실행 → 태그 변조 시 영향 큼)
  - 버전 갱신: Dependabot(`package-ecosystem: github-actions`) 활용 권장

## 프로젝트 README 기재 예시

```markdown
## 릴리스

- [release-please](https://github.com/googleapis/release-please) 기반 버전 관리 (`.github/workflows/release-please.yml`)
- PR 제목 [Conventional Commits](https://www.conventionalcommits.org/ko/v1.0.0/) 형식 필수 (squash 병합 커밋 = PR 제목)
  - `feat` → MINOR, 그 외 CHANGELOG 노출 접두어 → PATCH, `!` 표기 → MAJOR
  - `ci`, `chore`, `test`, `style`: CHANGELOG 제외, 단독으로는 릴리스 PR 미생성
- 릴리스 흐름
  1. `main` 병합 시 릴리스 PR(`chore(main): release vX.Y.Z`) 자동 생성/갱신
  2. 릴리스 PR 병합 시 태그(`vX.Y.Z`) 및 GitHub Release 자동 생성
- 설정: `release-please-config.json`, 현재 버전: `.release-please-manifest.json`
```
