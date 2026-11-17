# 확장 개요

Spec Kit 확장을 사용해 내장 스펙 주도 개발 프로세스를 넘어서는 명령, 외부 도구
연동, 품질 게이트, 그 밖의 기능을 추가하는 방법에 대한 가이드입니다.

## 목차

1. [소개](#introduction)
2. [시작하기](#getting-started)
3. [확장 찾기](#finding-extensions)
4. [확장 설치하기](#installing-extensions)
5. [확장 사용하기](#using-extensions)
6. [확장 관리하기](#managing-extensions)
7. [설정](#configuration)
8. [문제 해결](#troubleshooting)
9. [모범 사례](#best-practices)

---

<a id="introduction"></a>

## 소개

### 확장이란 무엇인가요?

확장은 코어 프레임워크를 비대하게 만들지 않고 Spec Kit에 새 명령과 기능을 추가하는 모듈식 패키지입니다. 확장으로 다음을 할 수 있습니다.

- 외부 도구(Jira, Linear, GitHub 등)와 **연동**
- 훅으로 반복 작업 **자동화**
- 팀에 맞게 워크플로 **사용자 정의**
- 프로젝트 간에 해결책 **공유**

### 왜 확장을 사용하나요?

- **깔끔한 코어**: spec-kit을 가볍고 집중된 상태로 유지합니다
- **선택적 기능**: 필요한 것만 설치합니다
- **커뮤니티 주도**: 누구나 확장을 만들고 공유할 수 있습니다
- **버전 관리**: 확장은 독립적으로 버전이 관리됩니다

<a id="specification-artifact-maintenance"></a>

### 명세 아티팩트 유지 관리

확장은 기능 전달 중이나 이후에 `spec.md`, `plan.md`, `tasks.md`를 조정하는 명령이나
훅을 정의할 수 있습니다. 그런 편집은 확장의 문서화된 동작과 프로젝트의 거버넌스를
따릅니다. 코어 Spec Kit 명령이 수행하는 것도, 그 책임에 추가되는 것도 아닙니다.
전달 이후 아티팩트를 어떻게 유지 관리할지는 팀의 보존 모델이 별도로 결정합니다.

---

<a id="getting-started"></a>

## 시작하기

### 사전 요구 사항

- 확장의 `requires.speckit_version` 제약과 호환되는 현재 Spec Kit 릴리스
- `specify init`으로 초기화된 프로젝트(`.specify/` 폴더가 있는 디렉터리)

### 버전 확인

```bash
specify version
```

### 첫 확장

번들된 GitHub 확장으로 시작하세요.

```bash
# 1. 확장 검색
specify extension search github

# 2. 상세 정보 확인
specify extension info github

# 3. 설치
specify extension add github
```

이 확장에는 Git과 설정된 GitHub MCP 서버가 필요합니다. 그것들을 사용할 수 있게
되면 GitHub Copilot 채팅에서 `/speckit-github-taskstoissues`를 호출해 기능의
`tasks.md`를 GitHub 이슈로 변환하세요. 이는 터미널 명령이 아니라 채팅 명령입니다.

아래의 Jira 예시는 별도로 검증한 서드파티 확장을 보여 줍니다. 검토한
아카이브에서 설치했거나 직접 신뢰하는 카탈로그를 설정했다고 가정합니다. Jira는
내장 커뮤니티 카탈로그에서 탐색 전용입니다.

---

<a id="finding-extensions"></a>

## 확장 찾기

`specify extension search`는 기본적으로 커뮤니티 카탈로그를 포함한 **모든 활성 카탈로그**를 동시에 검색합니다. 결과에는 소스 카탈로그와 설치 상태가 표시됩니다.

커뮤니티 카탈로그는 탐색 전용입니다. 직접 설치하거나 자신의 신뢰 카탈로그에
추가하기 전에 릴리스를 검증하세요.
[카탈로그 신뢰 모델](catalogs.md#trust-model-discovery-only-vs-install-sources)을
참고하세요.

### 모든 확장 둘러보기

```bash
specify extension search
```

모든 활성 카탈로그(기본적으로 default와 community)의 모든 확장을 보여 줍니다.

### 키워드로 검색

```bash
# "jira" 검색
specify extension search jira

# "issue tracking" 검색
specify extension search issue
```

### 태그로 필터링

```bash
# 모든 이슈 트래킹 확장 찾기
specify extension search --tag issue-tracking

# 모든 Atlassian 도구 찾기
specify extension search --tag atlassian
```

### 작성자로 필터링

```bash
# Stats Perform의 확장
specify extension search --author "Stats Perform"
```

### 검증된 것만 표시

```bash
# 검증된 확장만 표시
specify extension search --verified
```

### 확장 세부 정보 확인

```bash
# 상세 정보
specify extension info jira
```

다음을 보여 줍니다.

- 설명
- 요구 사항
- 제공되는 명령
- 사용 가능한 훅
- 링크(문서, 저장소, 변경 로그)
- 설치 상태

---

<a id="installing-extensions"></a>

## 확장 설치하기

### 카탈로그에서 설치

```bash
# 번들된 확장을 이름으로 설치
specify extension add github
```

설치가 활성화된 카탈로그가 공개한 확장의 경우, 설치는 다음을 수행합니다.

1. 공개된 아카이브를 다운로드(번들된 확장은 함께 제공된 사본을 사용)
2. 매니페스트 검증
3. spec-kit 버전과의 호환성 확인
4. `.specify/extensions/<extension-id>/`에 설치
5. 코딩 에이전트에 명령 등록
6. 기존 설정을 덮어쓰지 않고 선언된 설정 템플릿을 스캐폴딩

Jira를 검증한 뒤에는 여러분이 통제하는 카탈로그가 이를 설치 가능한 것으로 공개할
수 있습니다.

```bash
specify extension add jira
```

### URL에서 설치

```bash
# GitHub 릴리스에서
specify extension add <extension-name> --from https://github.com/org/spec-kit-ext/archive/refs/tags/v1.0.0.zip
```

### 로컬 디렉터리에서 설치(개발용)

```bash
# 테스트 또는 개발용
specify extension add --dev /path/to/extension
```

### 특정 카탈로그 버전 설치

```bash
specify extension info jira --versions
specify extension add jira --version <version>
```

요청한 버전은 이기는 설치 활성화 카탈로그 항목이 공개한 것이어야 합니다. 선택
규칙과 옵션 제한은 [CLI 레퍼런스](cli.md#install-an-extension)를 참고하세요.

### 설치 출력

검증한 소스에서 설치한 Jira 릴리스의 예시 출력입니다.

```text
✓ Extension installed successfully!

Jira Integration (v1.0.0)
  Create Jira Epics, Stories, and Issues from spec-kit artifacts

Provided commands:
  • speckit-jira-specstoissues - Create Jira hierarchy from spec and tasks
  • speckit-jira-discover-fields - Discover Jira custom fields for configuration
  • speckit-jira-sync-status - Sync task completion status to Jira

⚠  Configuration may be required
   Check: .specify/extensions/jira/
```

---

<a id="using-extensions"></a>

## 확장 사용하기

### 확장 명령 사용

확장은 GitHub Copilot에 나타나는 스킬을 추가합니다.

```text
# In GitHub Copilot
> /speckit-jira-specstoissues
```

### 확장 설정

대부분의 확장은 설정이 필요합니다.

```bash
# 1. 설정 파일 찾기
ls .specify/extensions/jira/

# 2. 템플릿을 설정으로 복사
cp .specify/extensions/jira/jira-config.template.yml \
   .specify/extensions/jira/jira-config.yml

# 3. 설정 편집
vim .specify/extensions/jira/jira-config.yml

# 4. 확장 사용
# (이제 명령이 여러분의 설정으로 동작합니다)
```

### 확장 훅

일부 확장은 코어 명령 이후에 실행되는 훅을 제공합니다.

**예시**: Jira 확장은 `/speckit-tasks`에 훅을 겁니다

```text
# Run core command
> /speckit-tasks

# Output includes:
## Extension Hooks

**Optional Hook**: jira
Command: `/speckit-jira-specstoissues`
Description: Automatically create Jira hierarchy after task generation

Prompt: Create Jira issues from tasks?
To execute: `/speckit-jira-specstoissues`
```

그런 다음 훅을 실행할지 건너뛸지 선택할 수 있습니다.

필수 훅은 자동 명령으로 드러납니다. 현재 명령 템플릿은 설정된 YAML 순서를
사용하고, 비활성화된 훅과 비어 있지 않은 조건을 가진 훅을 건너뛰며, 예약된
`auto_execute_hooks` 설정을 참조하지 않습니다.
자세한 내용은 [훅 설정 계약](api.md#hook-configuration)을 참고하세요.

확장은 세션 시작이나 도구 사용 같은 코딩 에이전트 라이프사이클 이벤트에도 반응할
수 있습니다. 훅과 달리 런타임 이벤트는 코딩 에이전트에 Markdown 지침을 보내는 대신
스크립트를 직접 실행합니다.
[확장 런타임 이벤트](api.md#extension-runtime-events)를 참고하세요.

---

<a id="managing-extensions"></a>

## 확장 관리하기

### 설치된 확장 나열

```bash
specify extension list
```

출력:

```text
Installed Extensions:

  ✓ Jira Integration (v1.0.0)
     Create Jira Epics, Stories, and Issues from spec-kit artifacts
     Commands: 3 | Hooks: 1 | Status: Enabled
```

### 확장 업데이트

```bash
# 업데이트 확인(모든 확장)
specify extension update

# 특정 확장 업데이트
specify extension update jira
```

출력:

```text
🔄 Checking for updates...

Updates available:

  • jira: 1.0.0 → 1.1.0

Update these extensions? [y/N]:
```

### 확장을 일시적으로 비활성화

```bash
# 제거하지 않고 비활성화
specify extension disable jira
```

선언적 훅은 `.specify/extensions.yml`에서 비활성으로 표시됩니다. 그런 다음 네이티브
런타임 이벤트 설정이 최선의 노력으로 새로 고쳐집니다. 새로 고침 실패는 경고를
만들지만, 오래된 네이티브 훅이 활성 상태로 남아 실행될 수 있습니다. 영향받는 통합에
대해 `specify integration upgrade <key>`로 다시 시도하세요.
확장은 `specify extension enable jira`로 다시 활성화합니다.

generic 통합의 경우 비활성화는 소유한 명령 또는 스킬 파일도 제거합니다. 다른
통합은 통합 전환, 제거, 조정이 그것을 제거할 때까지 호출 가능한 상태로 남는 생성
파일을 유지할 수 있습니다.
비활성화된 확장은 레지스트리 기반 아티팩트 해석과 향후의 통합 등록에서
제외됩니다. 비활성화는 보편적인 파일 제거나 실행 차단 메커니즘이 아닙니다.

### 확장 다시 활성화

```bash
specify extension enable jira

✓ Extension 'jira' enabled
```

### 확장 제거

```bash
# 확장 제거(확인 포함)
specify extension remove jira

# 제거할 때 설정 유지
specify extension remove jira --keep-config

# 강제 제거(확인 없음)
specify extension remove jira --force
```

---

<a id="configuration"></a>

## 설정

### 설정 파일

확장은 여러 설정 파일을 가질 수 있습니다.

```text
.specify/extensions/jira/
├── jira-config.yml           # Main config (version controlled)
├── local-config.yml         # Local overrides (gitignored)
└── jira-config.template.yml  # Template (reference)
```

### 설정 레이어

설정은 다음 순서로 병합됩니다(마지막이 가장 높은 우선순위).

1. **확장 기본값**(최상위 `defaults`가 `config.defaults`와 병합되며, 충돌 시
   `config.defaults`가 이김)
2. **프로젝트 설정** (`jira-config.yml`)
3. **로컬 오버라이드** (`local-config.yml`)
4. **환경 변수** (`SPECKIT_JIRA_*`)

### 예시: Jira 설정

**프로젝트 설정** (`.specify/extensions/jira/jira-config.yml`):

```yaml
project:
  key: "MSATS"

defaults:
  epic:
    labels: ["spec-driven"]
```

**로컬 오버라이드** (`.specify/extensions/jira/local-config.yml`):

```yaml
project:
  key: "MYTEST"  # Override for local development
```

**환경 변수**:

```bash
export SPECKIT_JIRA_PROJECT_KEY="DEVTEST"
```

확장이 `ConfigManager`나 그에 상응하는 로딩을 사용하면, 최종 해석된 설정은 환경
변수의 `DEVTEST`를 사용합니다. Spec Kit은 병합된 설정을 임의의 명령이나 스크립트에
자동으로 주입하지 않습니다. 확장이 그것을 로드해야 합니다. YAML의 `${...}`
문자열은 `ConfigManager`가 보간하지 않습니다.
[설정 로딩](developing.md#config-loading)을 참고하세요.

### 프로젝트 전체 확장 설정

파일: `.specify/extensions.yml`

```yaml
# Extensions installed in this project
installed:
  - jira
  - linear

# Global settings
settings:
  auto_execute_hooks: true  # Reserved; currently not consulted

# Hook configuration
# Available events: before_specify, after_specify, before_plan, after_plan,
#                   before_tasks, after_tasks, before_implement, after_implement,
#                   before_analyze, after_analyze, before_checklist, after_checklist,
#                   before_clarify, after_clarify, before_constitution, after_constitution,
# Bundled bug extension events (when installed):
#                   before_bug_assess, after_bug_assess, before_bug_fix, after_bug_fix,
#                   before_bug_test, after_bug_test
hooks:
  after_tasks:
    - extension: jira
      command: speckit.jira.specstoissues
      enabled: true
      optional: true
      prompt: "Create Jira issues from tasks?"
```

모든 필드와 현재의 실행 제한 사항은
[훅 설정 계약](api.md#hook-configuration)을 참고하세요.

---

<a id="troubleshooting"></a>

## 문제 해결

### 확장을 찾을 수 없음

**오류**: `Extension 'jira' not found in catalog`

**해결책**:

1. 철자 확인: `specify extension search jira`
2. 활성 카탈로그 확인: `specify extension catalog list`
3. 인터넷 연결 확인
4. 확장이 아직 게시되지 않았을 수 있음

`specify extension search --help`는 도움말을 표시할 뿐, 캐시된 카탈로그 데이터를
새로 고치지 않습니다. 카탈로그 캐시는 보통 한 시간 후 만료됩니다. Python 호출자는
`ExtensionCatalog.clear_cache()`로 명시적으로 지울 수 있습니다.

### 설정을 찾을 수 없음

**오류**: `Jira configuration not found`

**해결책**:

1. 확장이 설치되어 있는지 확인: `specify extension list`
2. 템플릿으로부터 설정 생성:

   ```bash
   cp .specify/extensions/jira/jira-config.template.yml \
      .specify/extensions/jira/jira-config.yml
   ```

3. 확장 재설치: `specify extension remove jira && specify extension add jira`

### 명령을 사용할 수 없음

**문제**: 확장 명령이 코딩 에이전트에 나타나지 않음

**해결책**:

1. 확장이 활성화되어 있는지 확인: `specify extension list`
2. GitHub Copilot 다시 시작
3. 스킬 파일이 존재하는지 확인:

   ```bash
   ls .github/skills/speckit-jira-*/SKILL.md
   ```

4. 확장 재설치

### 호환되지 않는 버전

**오류**: `Extension requires spec-kit >=0.2.0, but you have 0.1.0`

**해결책**:

1. spec-kit 업그레이드:

   ```bash
   uv tool upgrade specify-cli
   ```

2. 확장의 이전 버전 설치:

   ```bash
   specify extension add <extension-name> --from https://github.com/org/ext/archive/v1.0.0.zip
   ```

### MCP 도구를 사용할 수 없음

**오류**: `Tool 'jira-mcp-server/epic_create' not found`

**해결책**:

1. MCP 서버가 설치되어 있는지 확인
2. 코딩 에이전트의 MCP 설정 확인
3. 코딩 에이전트 다시 시작
4. 확장 요구 사항 확인: `specify extension info jira`

### 권한 거부됨

**오류**: Jira에 접근할 때 `Permission denied`

**해결책**:

1. MCP 서버 설정의 Jira 자격 증명 확인
2. Jira의 프로젝트 권한 확인
3. MCP 서버 연결을 독립적으로 테스트

---

<a id="best-practices"></a>

## 모범 사례

### 1. 버전 관리

**커밋할 것**:

- `.specify/extensions.yml` (프로젝트 확장 설정)
- `.specify/extensions/*/jira-config.yml` (프로젝트 설정)

**커밋하지 말 것**:

- `.specify/extensions/.cache/` (카탈로그 캐시)
- `.specify/extensions/.backup/` (설정 백업)
- `.specify/extensions/*/local-config.yml` (로컬 오버라이드)
- `.specify/extensions/.registry` (설치 상태)

`.gitignore`에 추가하세요.

```gitignore
.specify/extensions/.cache/
.specify/extensions/.backup/
.specify/extensions/*/local-config.yml
.specify/extensions/.registry
```

### 2. 팀 워크플로

**팀의 경우**:

1. 어떤 확장을 사용할지 합의
2. 확장 설정 커밋
3. README에 확장 사용법 문서화
4. 확장을 함께 최신 상태로 유지

**README 섹션 예시**:

```markdown
## Extensions

This project uses:
- **jira** (v1.0.0) - Jira integration
  - Config: `.specify/extensions/jira/jira-config.yml`
  - Requires: jira-mcp-server

To install: `specify extension add jira`
```

### 3. 로컬 개발

개발에는 로컬 설정을 사용하세요.

```yaml
# .specify/extensions/jira/local-config.yml
project:
  key: "DEVTEST"  # Your test project

defaults:
  task:
    custom_fields:
      customfield_10002: 1  # Lower story points for testing
```

### 4. 환경별 설정

CI/CD에는 환경 변수를 사용하세요.

```bash
# .github/workflows/deploy.yml
env:
  SPECKIT_JIRA_PROJECT_KEY: ${{ secrets.JIRA_PROJECT }}

- name: Create Jira Issues
  run: specify extension add jira && ...
```

### 5. 확장 업데이트

**정기적으로 업데이트 확인**:

```bash
# 매주 또는 주요 릴리스 전에
specify extension update
```

**재현 가능한 설치를 위해 정확한 카탈로그 버전 선택**:

```bash
specify extension add jira --version 1.0.0
```

팀을 위해 설치 명령을 기록해 두세요. `.specify/extensions.yml`의 `installed`
목록은 버전 잠금 메커니즘이 아닙니다. `update`를 명시적으로 실행하면 더 새로운
버전이 설치될 수 있습니다.

### 6. 최소한의 확장

실제로 사용하는 확장만 설치하세요.

- 복잡성 감소
- 더 빠른 명령 로딩
- 더 적은 설정

### 7. 문서화

프로젝트에 확장 사용법을 문서화하세요.

```markdown
# PROJECT.md

## Working with Jira

After creating tasks, sync to Jira:
1. Run `/speckit-tasks` to generate tasks
2. Run `/speckit-jira-specstoissues` to create Jira issues
3. Run `/speckit-jira-sync-status` to update status
```

---

## FAQ

### Q: 여러 확장을 동시에 사용할 수 있나요?

**A**: 네! 확장은 함께 동작하도록 설계되었습니다. 필요한 만큼 설치하세요.

### Q: 확장이 spec-kit을 느리게 만드나요?

**A**: 확장 메타데이터는 목록 조회, 설치, 아티팩트 해석 같은 작업 중에 읽힙니다.
훅은 Spec Kit 명령 전후에 작업을 추가할 수 있고, 런타임 이벤트는 코딩 에이전트
라이프사이클 이벤트 중에 스크립트를 실행할 수 있습니다. 그 비용은 설치된 확장과
설정된 동작에 따라 달라집니다.

### Q: 비공개 확장을 만들 수 있나요?

**A**: 네. `--dev`나 `--from`으로 설치하고 비공개로 유지하세요. 공개 카탈로그 제출은 선택 사항입니다.

### Q: 확장이 안전한지 어떻게 아나요?

**A**: verified 배지는 코드 감사 보증이 아닙니다. Spec Kit 메인테이너는
독립적으로 유지 관리되는 확장 코드를 검토, 감사, 보증, 지원하지 않습니다.
설치하기 전에 소스와 릴리스 아카이브를 직접 검증해야 합니다. 확장은 여러분의
판단에 따라 사용하세요.

### Q: 확장이 spec-kit 코어를 수정할 수 있나요?

**A**: 확장은 Spec Kit의 소스 코드를 바꾸지 않고 명령, 템플릿, 스크립트, 훅,
런타임 이벤트를 기여합니다. 그 명령과 훅은 문서화된 동작과 프로젝트의 거버넌스
아래에서 프로젝트 파일을 편집할 수 있습니다.
[명세 아티팩트 유지 관리](#specification-artifact-maintenance)를 참고하세요.

### Q: 두 확장이 같은 명령 이름을 가지면 어떻게 되나요?

**A**: 네임스페이스가 붙은 명령(`speckit.{extension}.{command}`)이 충돌을
줄입니다. 설치는 설치된 확장과의 정확한 명령 이름 또는 별칭 충돌, 그리고 코어
명령과의 정규화된 출력 이름 충돌을 거부합니다.
해석 우선순위는 템플릿 같은 기여 아티팩트의 우선순위를 결정합니다. 충돌하는 명령
이름이 설치되는 것을 허용하지는 않습니다.
아티팩트 해석 우선순위를 바꾸려면
[`specify extension set-priority`](cli.md#set-extension-priority)를 사용하세요.

### Q: 기존 확장에 기여할 수 있나요?

**A**: 네! 대부분의 확장은 오픈소스입니다. `specify extension info {extension}`의 저장소 링크를 확인하세요.

### Q: 확장 버그는 어떻게 보고하나요?

**A**: 확장의 저장소(`specify extension info`에 표시됨)로 가서 이슈를 만드세요.

### Q: 확장이 오프라인에서 동작하나요?

**A**: 일단 설치되면 확장은 오프라인에서 동작합니다. 다만 일부 확장은 기능을 위해 인터넷이 필요할 수 있습니다(예: Jira는 Jira API 접근이 필요).

### Q: 확장 설정은 어떻게 백업하나요?

**A**: 확장 설정은 `.specify/extensions/{extension}/`에 있습니다. 이 디렉터리를 백업하거나 설정을 git에 커밋하세요.

---

## 지원

- **확장 문제**: 확장 저장소에 보고(`specify extension info` 참고)
- **Spec Kit 문제**: <https://github.com/github/spec-kit/issues>
- **확장 카탈로그**: <https://github.com/github/spec-kit/tree/main/extensions>
- **문서**: [확장 개발하기](developing.md)와 [확장 게시 가이드](publishing.md) 참고

---

*최종 업데이트: 2026-01-28*
*Spec Kit 버전: 0.1.0*

> 이 페이지의 에이전트 호출 예시는 GitHub Copilot의 기본 스킬 구문(`/speckit-*`)을 사용합니다. 다른 통합과 모드는 [명령 호출](../integrations/cli.md#command-invocation)을 참고하세요.

## 함께 보기

- [확장 CLI 레퍼런스](cli.md)
- [확장 카탈로그](catalogs.md)
- [커뮤니티 확장](../community/extensions.md)
- [확장 개발하기](developing.md)
- [확장 API 레퍼런스](api.md)
- [확장 게시하기](publishing.md)
- [명세 보존하기](../how-to/persisting-specs.md)
