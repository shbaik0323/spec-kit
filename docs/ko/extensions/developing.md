# 확장 개발하기

GitHub Copilot의 스킬 레이아웃을 사용해 Spec Kit 확장을 만드는 가이드입니다.
Copilot 채팅에서 `/speckit-my-ext-hello` 같은 하이픈 이름으로 스킬을 호출하세요.
확장 매니페스트는 여전히 `speckit.my-ext.hello` 같은 점 표기 명령 식별자를
사용합니다. Spec Kit이 설치 중에 이를 Copilot 스킬로 렌더링합니다.

---

## 빠른 시작

### 1. 확장 디렉터리 만들기

```bash
mkdir my-extension
cd my-extension
```

### 2. `extension.yml` 매니페스트 만들기

```yaml
schema_version: "1.0"

extension:
  id: "my-ext"                          # Lowercase, alphanumeric + hyphens only
  name: "My Extension"
  version: "1.0.0"                      # SemVer recommended; PEP 440 accepted
  description: "My custom extension"
  author: "Your Name"
  repository: "https://github.com/you/spec-kit-my-ext"
  license: "MIT"

requires:
  speckit_version: ">=0.1.0"            # Minimum spec-kit version
  tools:                                # Optional: External tools required
    - name: "my-tool"
      required: true
      version: ">=1.0.0"
  commands:                             # Optional: Core commands needed
    - "speckit.tasks"

provides:
  commands:
    - name: "speckit.my-ext.hello"      # Must follow pattern: speckit.{ext-id}.{cmd}
      file: "commands/hello.md"
      description: "Say hello"

  config:                               # Optional: Config files
    - name: "my-ext-config.yml"
      template: "my-ext-config.template.yml"
      description: "Extension configuration"
      required: false

hooks:                                  # Optional: Integration hooks
  after_tasks:
    command: "speckit.my-ext.hello"
    optional: true
    prompt: "Run hello command?"

tags:                                   # Optional: For catalog search
  - "example"
  - "utility"
```

### 3. commands 디렉터리 만들기

```bash
mkdir commands
```

### 4. 명령 파일 만들기

**파일**: `commands/hello.md`

````markdown
---
description: "Say hello command"
tools:                              # Optional: AI tools this command uses
  - 'some-tool/function'
scripts:                            # Optional: Helper scripts
  sh: ../../scripts/bash/helper.sh
  ps: ../../scripts/powershell/helper.ps1
---

# Hello Command

This command says hello!

## User Input

$ARGUMENTS

## Steps

1. Greet the user
2. Show extension is working

```bash
echo "Hello from my extension!"
echo "Arguments: $ARGUMENTS"
```

## Extension Configuration

Load extension config from `.specify/extensions/my-ext/my-ext-config.yml`.
````

### 5. 로컬에서 테스트

```bash
cd /path/to/spec-kit-project
specify integration install copilot --integration-options="--skills"
specify extension add --dev /path/to/my-extension
```

새 프로젝트라면
`specify init my-project --integration copilot --integration-options="--skills"`로
초기화하세요.
Copilot이 이미 명령 레이아웃으로 설치되어 있다면 다시 설치하지 말고
[Copilot 마이그레이션 안내](../integrations/copilot.md#commands-layout)를
따르세요.

### 6. 설치 확인

```bash
specify extension list

# 다음과 같이 표시되어야 함:
#  ✓ My Extension (v1.0.0)
#     My custom extension
#     Commands: 1 | Hooks: 1 | Status: Enabled
```

### 7. 스킬 테스트

GitHub Copilot 채팅을 열고 스킬을 호출하세요.

```text
/speckit-my-ext-hello world
```

생성된 스킬은 `.github/skills/speckit-my-ext-hello/SKILL.md`에 있습니다. 이 호출은
터미널이 아니라 Copilot 채팅에서 하는 것입니다.

---

## 매니페스트 스키마 레퍼런스

### 필수 필드

#### `schema_version`

확장 매니페스트 스키마 버전. 현재: `"1.0"`

#### `extension`

확장 메타데이터 블록.

**필수 하위 필드**:

- `id`: 확장 식별자(소문자, 영숫자, 하이픈)
- `name`: 사람이 읽을 수 있는 이름
- `version`: PEP 440 버전(SemVer 권장, 예: "1.0.0")
- `description`: 짧은 설명

**선택적 하위 필드**:

- `author`: 확장 작성자
- `repository`: 소스 코드 URL
- `license`: SPDX 라이선스 식별자
- `homepage`: 확장 홈페이지 URL

#### `requires`

호환성 요구 사항.

**필수 하위 필드**:

- `speckit_version`: 시맨틱 버전 지정자(예: ">=0.1.0,<2.0.0")

**선택적 하위 필드**:

- `tools`: 필요한 외부 도구(도구 객체의 배열)
- `commands`: 필요한 코어 spec-kit 명령(명령 이름의 배열)
- `scripts`: 필요한 코어 스크립트(스크립트 이름의 배열)

#### `provides`

확장이 제공하는 것.

**선택적 하위 필드:**

- `commands`: 명령 객체의 배열
- `templates`: 템플릿 객체의 배열
- `scripts`: 스크립트 객체의 배열

`hooks`와 `events`는 별도의 최상위 매니페스트 필드입니다(`provides` 아래에 중첩되지
않고 그 형제입니다 — 아래의 [`hooks`](#hooks) 참고). `provides.commands`,
`provides.templates`, `provides.scripts`, `hooks`, `events` 중 하나 이상이
필요합니다.

**명령 객체**:

- `name`: 명령 이름(`speckit.{ext-id}.{command}`와 일치해야 함)
- `file`: 명령 파일의 경로(확장 루트 기준 상대 경로)
- `description`: 명령 설명(선택)

**템플릿 객체**:

- `name`: 템플릿 이름(소문자, 영숫자, 하이픈 — 예: `myext-template`)
- `file`: 템플릿 파일의 경로(확장 루트 기준 상대 경로)
- `description`: 템플릿 설명(선택)

**스크립트 객체**:

- `name`: 스크립트 이름(소문자, 영숫자, 하이픈 — 예: `myext-collect`)
- `file`: 스크립트 파일의 경로(확장 루트 기준 상대 경로)
- `description`: 스크립트 설명(선택)
- `runtimes`: 스크립트가 지원하는 런타임(선택, 배열. `bash`, `powershell`, `python`의 부분 집합 — 참고용일 뿐이며 스크립트를 선택하거나 호출하는 데 쓰이지 않음)

확장이 제공하는 템플릿과 스크립트는 항상 `replace`로 해석됩니다. 이런 항목에 `strategy` 키를 포함한 매니페스트는 `ValidationError`로 거부됩니다. 합성 가능한 전략(`wrap`/`prepend`/`append`)은 프리셋 전용입니다.

### 선택적 필드

<a id="hooks"></a>

#### `hooks`

자동 실행을 위한 통합 훅.

사용 가능한 훅 지점:

- `before_specify` / `after_specify`: 명세 생성 전/후
- `before_plan` / `after_plan`: 구현 계획 전/후
- `before_tasks` / `after_tasks`: 태스크 생성 전/후
- `before_implement` / `after_implement`: 구현 전/후
- `before_analyze` / `after_analyze`: 아티팩트 간 분석 전/후
- `before_checklist` / `after_checklist`: 체크리스트 생성 전/후
- `before_clarify` / `after_clarify`: 스펙 명확화 전/후
- `before_constitution` / `after_constitution`: 컨스티튜션 갱신 전/후
- `before_bug_assess` / `after_bug_assess`: 버그 평가 전/후(번들된 `bug` 확장)
- `before_bug_fix` / `after_bug_fix`: 버그 교정 전/후(번들된 `bug` 확장)
- `before_bug_test` / `after_bug_test`: 버그 검증 전/후(번들된 `bug` 확장)

각 이벤트는 단일 훅 객체 또는 훅 객체의 목록을 받습니다(한 이벤트에 여러 명령).

훅 객체:

- `command`: 실행할 명령(보통 `provides.commands`의 것이지만, 등록된 어떤 명령이든 참조 가능)
- `priority`: `HookExecutor`를 위한 정렬 메타데이터(1 이상의 정수, 기본값 10).
  현재 명령 템플릿은 대신 설정된 YAML 순서로 훅을 드러냅니다
- `optional`: true이면 실행 전에 사용자에게 묻습니다
- `prompt`: 선택적 훅의 프롬프트 텍스트
- `description`: 훅 설명
- `condition`: `HookExecutor`가 평가하는 표현식. 현재 명령 템플릿은 비어 있지 않은
  조건을 가진 훅을 평가하지 않고 건너뜁니다

모든 필드와 Python API 및 명령 템플릿 동작 사이의 구분은
[훅 설정 계약](api.md#hook-configuration)을 참고하세요.

#### `events`

런타임 이벤트는 Spec Kit 명령이 아니라 코딩 에이전트의 라이프사이클에 반응합니다.
최상위에 정식 이름을 선언하고 실행 가능한 `scripts` 항목이 있는 명령 템플릿을
참조하세요.

```yaml
events:
  session_start:
    command: speckit.my-ext.initialize
```

예를 들어 `commands/initialize.md`는 다음을 선언할 수 있습니다.

```yaml
---
description: "Initialize the extension session"
scripts:
  sh: scripts/initialize.sh
  ps: scripts/initialize.ps1
---
```

그 명령을 `provides.commands`에 포함하고 확장에 스크립트를 제공하세요. 런타임
이벤트는 스크립트를 직접 실행합니다. 명령의 Markdown 본문을 코딩 에이전트에 보내지
않습니다. 각 통합은 정식 이벤트의 일부를 지원합니다.
[런타임 이벤트 계약](api.md#extension-runtime-events)을 참고하세요.

#### `tags`

카탈로그 탐색을 위한 태그 배열.

#### `defaults`

확장 설정 기본값.

---

## 명령 파일 형식

### 프런트매터(YAML)

```yaml
---
description: "Command description"          # Recommended
tools:                                      # Optional
  - 'tool-name/function'
scripts:                                    # Optional
  sh: ../../scripts/bash/helper.sh
  ps: ../../scripts/powershell/helper.ps1
---
```

### 본문(Markdown)

특수 자리 표시자와 함께 표준 Markdown을 사용하세요.

- `$ARGUMENTS`: 사용자가 제공한 인자
- `{SCRIPT}`: 등록 중에 스크립트 경로로 대체됨
- `__SPECKIT_COMMAND_<NAME>__`: 활성 통합의 구분자를 사용해 렌더링된, 다른 명령의 호출로 대체됨([다른 명령 참조하기](#referencing-other-commands) 참고)

**예시**:

````markdown
## Steps

1. Parse arguments
2. Execute logic

```bash
args="$ARGUMENTS"
echo "Running with args: $args"
```
````

<a id="referencing-other-commands"></a>

### 다른 명령 참조하기

명령 본문은 Spec Kit이 Copilot 스킬로 렌더링하는 *템플릿*입니다.
그 본문에서 다른 명령을 참조할 때는 **호출을 하드코딩하지 마세요**. Spec Kit이
올바른 스킬 구문을 렌더링하고 확장이 이식 가능하게 유지되도록 명령 참조 토큰을
사용하세요.

토큰 `__SPECKIT_COMMAND_<NAME>__`을 사용하세요. Copilot의 스킬 레이아웃에서 Spec
Kit은 이를 `/speckit-...` 호출로 해석합니다.

명령 이름은 `speckit.` 접두사를 떼고 점으로 구분된 각 세그먼트 구분자를 밑줄로 바꿔 대문자로 인코딩하세요. 세그먼트 안의 하이픈은 하이픈으로 유지됩니다.

| 명령 파일 | 토큰 |
| --- | --- |
| `speckit.plan.md` | `__SPECKIT_COMMAND_PLAN__` |
| `speckit.bug.fix.md` | `__SPECKIT_COMMAND_BUG_FIX__` |
| `speckit.git.commit.md` | `__SPECKIT_COMMAND_GIT_COMMIT__` |
| `speckit.agent-context.update.md` | `__SPECKIT_COMMAND_AGENT-CONTEXT_UPDATE__` |

밑줄은 명령 세그먼트를 구분하고 하이픈은 그것이 속한 세그먼트의 일부이므로,
`AGENT-CONTEXT_UPDATE`는 세 개가 아니라 `agent-context`와 `update` 두
세그먼트입니다. Copilot은 이를 `/speckit-agent-context-update`로 렌더링합니다.

**예시** — 사용자에게 다음 단계를 안내하는 명령 본문:

```markdown
Once the assessment exists, the next step is `__SPECKIT_COMMAND_BUG_FIX__ slug=<slug>`.
```

이는 Copilot 채팅에서 `/speckit-bug-fix slug=<slug>`로 렌더링됩니다. 퍼스트파티
`bug`와 `git` 확장은 이 토큰만 사용합니다. 동작하는 예시는
`extensions/bug/commands/`를 참고하세요.

다른 통합의 호출 형식은 공통
[명령 호출 레퍼런스](../integrations/cli.md#command-invocation)를 참고하세요.

### 스크립트 경로 재작성

등록기는 명령을 렌더링하면서 헬퍼 경로를 정규화합니다. `../` 세그먼트가 둘 이상인
저장소 루트 참조는 프로젝트의 코어 스크립트를 대상으로 합니다.

**확장에서**:

```yaml
scripts:
  sh: ../../scripts/bash/helper.sh
```

선택된 스크립트 변형을 호출하려면 명령 본문에서 `{SCRIPT}`를 사용하세요.
이 예시에서 생성되는 호출은 다음과 같습니다.

```bash
.specify/scripts/bash/helper.sh
```

반면 단독 `scripts/helper.sh`는 확장 소유 스크립트인
`.specify/extensions/<extension-id>/scripts/helper.sh`를 대상으로 합니다. 생성된
출력 형식은 `{SCRIPT}`를 해석한 뒤 원래의 `scripts` 프런트매터를 생략할 수
있습니다. 설치된 명령이나 스킬에 그 메타데이터가 남아 있을 것이라고 기대하지
마세요.

---

## 설정 파일

### 설정 템플릿

**파일**: `my-ext-config.template.yml`

```yaml
# My Extension Configuration
# Copy this to my-ext-config.yml and customize

# Example configuration
api:
  endpoint: "https://api.example.com"
  timeout: 30
  key: ""  # Set with SPECKIT_MY_EXT_API_KEY; do not commit credentials

features:
  feature_a: true
  feature_b: false
```

<a id="config-loading"></a>

### 설정 로딩

명령에서 계층적 우선순위로 설정을 로드하세요.

1. 확장 기본값(`extension.yml` → `defaults`, 충돌하는 키는 그다음
   `config.defaults`)
2. 프로젝트 설정(`.specify/extensions/my-ext/my-ext-config.yml`)
3. 로컬 오버라이드(`.specify/extensions/my-ext/local-config.yml` - gitignore 처리됨)
4. 환경 변수(`SPECKIT_MY_EXT_*`)

`specify-cli`가 설치된 환경에서 실행되는 Python 코드라면, 병합을 다시 구현하지
말고 기존 로더를 사용하세요.

```python
from pathlib import Path
from specify_cli.extensions import ConfigManager

config = ConfigManager(Path.cwd(), "my-ext").get_config()
endpoint = config.get("api", {}).get("endpoint")
```

프로젝트 루트에서 실행하거나 그 루트를 명시적으로 전달하세요. 환경 변수의 구성
요소는 중첩 키가 됩니다. `SPECKIT_MY_EXT_API_KEY`는 `api.key`를 재정의합니다. 값은
문자열로 남으며, YAML의 `${...}` 자리 표시자는 확장되지 않습니다.
다른 언어로 구현된 명령은 이 계층화가 필요하다면 자체 로딩을 제공해야 합니다.

Spec Kit은 설정을 검증하는 데 `config_schema`를 사용하지 않습니다.
필수 값과 타입은 확장의 명령이나 스크립트에서 검증하세요.

---

## `.extensionignore`로 파일 제외하기

확장 작성자는 확장 루트에 `.extensionignore` 파일을 만들어, 사용자가 `specify extension add`로 확장을 설치할 때 복사되지 않을 파일과 폴더를 제외할 수 있습니다. 개발 전용 파일(테스트, CI 설정, 문서 소스 등)을 설치된 사본에서 빼는 데 유용합니다.

### 형식

파일은 [`pathspec`](https://pypi.org/project/pathspec/) 라이브러리가 처리하는 `.gitignore` 호환 패턴(한 줄에 하나)을 사용합니다.

- 빈 줄은 무시됩니다
- `#`로 시작하는 줄은 주석입니다
- `*`는 `/`를 **제외한** 모든 것과 일치합니다(디렉터리 경계를 넘지 않음)
- `**`는 0개 이상의 디렉터리와 일치합니다(예: `docs/**/*.draft.md`)
- `?`는 `/`를 제외한 임의의 한 문자와 일치합니다
- 끝의 `/`는 패턴을 디렉터리로만 제한합니다
- `/`를 포함한 패턴(끝의 슬래시 제외)은 확장 루트에 고정됩니다
- `/`가 없는 패턴은 트리의 어느 깊이에서든 일치합니다
- `!`는 이전에 제외된 패턴을 부정합니다(파일을 다시 포함)
- 패턴의 백슬래시는 크로스 플랫폼 호환성을 위해 슬래시로 정규화됩니다
- `.extensionignore` 파일 자체는 항상 자동으로 제외됩니다

### 예시

```gitignore
# .extensionignore

# Development files
tests/
.github/
.gitignore

# Build artifacts
__pycache__/
*.pyc
dist/

# Documentation source (keep only the built README)
docs/
CONTRIBUTING.md
```

### 패턴 매칭

| 패턴 | 일치 | 일치하지 않음 |
|---------|---------|----------------|
| `*.pyc` | 모든 디렉터리의 모든 `.pyc` 파일 | — |
| `tests/` | `tests` 디렉터리(와 그 모든 내용) | `tests`라는 이름의 파일 |
| `docs/*.draft.md` | `docs/api.draft.md`(`docs/` 바로 안) | `docs/sub/api.draft.md`(중첩) |
| `.env` | 모든 수준의 `.env` 파일 | — |
| `!README.md` | 앞선 패턴에 일치했더라도 `README.md`를 다시 포함 | — |
| `docs/**/*.draft.md` | `docs/api.draft.md`, `docs/sub/api.draft.md` | — |

### 지원되지 않는 기능

다음 `.gitignore` 기능은 이 맥락에서 **적용되지 않습니다**.

- **여러 `.extensionignore` 파일**: 확장 루트의 단일 파일만 지원됩니다(`.gitignore`는 하위 디렉터리의 파일을 지원)
- **`$GIT_DIR/info/exclude`와 `core.excludesFile`**: Git 전용이며 여기에는 대응하는 것이 없습니다
- **제외된 디렉터리 안에서의 부정**: 파일 복사가 `shutil.copytree`를 사용하므로, 디렉터리를 제외하면 그 안으로의 재귀가 완전히 막힙니다. 부정 패턴은 그 자체가 제외된 디렉터리 안의 파일을 다시 포함시킬 수 없습니다. 예를 들어 `tests/` 다음에 `!tests/important.py`를 쓰는 조합은 `tests/important.py`를 보존하지 **않습니다**. `tests/` 디렉터리가 루트 수준에서 건너뛰어지고 그 내용은 평가되지 않습니다. 이를 우회하려면 디렉터리 자체가 아니라 그 내용을 개별적으로 제외하세요(예: `tests/` 대신 `tests/*.pyc`와 `tests/.cache/`).

---

## 검증 규칙

### 확장 ID

- **패턴**: `^[a-z0-9-]+$`
- **유효**: `my-ext`, `tool-123`, `awesome-plugin`
- **유효하지 않음**: `MyExt`(대문자), `my_ext`(밑줄), `my ext`(공백)

### 확장 버전

- **형식**: PEP 440. 릴리스에는 SemVer(`MAJOR.MINOR.PATCH`)를 권장
- **유효**: `1.0.0`, `0.1.0`, `2.5.3`
- **CLI가 받아들이는 것**: `1.0`, `v1.0.0`, `1.0.0-beta`
- **유효하지 않음**: `latest`, `not-a-version`

커뮤니티 제출에는 더 엄격한 릴리스 버전 요구 사항이 있습니다.
[제출 검증](publishing.md#submission-validation)을 참고하세요.

### 명령 이름

- **패턴**: `^speckit\.[a-z0-9-]+\.[a-z0-9-]+$`
- **유효**: `speckit.my-ext.hello`, `speckit.tool.cmd`
- **유효하지 않음**: `other-ext.hello`(잘못된 네임스페이스), `speckit.my-ext.Create`(대문자)

레거시 `my-ext.hello`와 `speckit.hello`는 경고와 함께 `speckit.my-ext.hello`로
정규화됩니다. 새 매니페스트에서는 정식 이름을 사용하세요.

### 명령 파일 경로

- 확장 루트 기준 상대 경로**여야 합니다**
- **유효**: `commands/hello.md`, `commands/subdir/cmd.md`
- **유효하지 않음**: `/absolute/path.md`, `../outside.md`

---

## 확장 테스트하기

### 수동 테스트

1. [Copilot 스킬](../integrations/copilot.md#setup)용으로 설정된 프로젝트에
   **테스트 확장을 만듭니다**.
2. **로컬로 설치합니다**.

   ```bash
   specify extension add --dev /path/to/extension
   ```

3. **설치를 확인합니다**.

   ```bash
   specify extension list
   ```

4. Copilot 채팅에서 **스킬을 테스트합니다**. 예: `/speckit-my-ext-hello world`.
5. **스킬 등록을 확인합니다**.

   ```bash
   ls .github/skills/speckit-my-ext-*/SKILL.md
   ```

6. **확장을 제거합니다**.

   ```bash
   specify extension remove my-ext
   ```

### 자동 테스트

확장에 대한 테스트를 만드세요.

```python
# tests/test_my_extension.py
import pytest
from pathlib import Path
from specify_cli.extensions import ExtensionManifest

def test_manifest_valid():
    """Test extension manifest is valid."""
    manifest = ExtensionManifest(Path("extension.yml"))
    assert manifest.id == "my-ext"
    assert len(manifest.commands) >= 1

def test_command_files_exist():
    """Test all command files exist."""
    manifest = ExtensionManifest(Path("extension.yml"))
    for cmd in manifest.commands:
        cmd_file = Path(cmd["file"])
        assert cmd_file.exists(), f"Command file not found: {cmd_file}"
```

---

## 배포

### 방법 1: GitHub 저장소

1. **저장소 만들기**: `spec-kit-my-ext`
2. **파일 추가**:

   ```text
   spec-kit-my-ext/
   ├── extension.yml
   ├── commands/
   ├── scripts/
   ├── docs/
   ├── README.md
   ├── LICENSE
   └── CHANGELOG.md
   ```

3. **릴리스 만들기**: 버전으로 태그 지정(예: `v1.0.0`)
4. **저장소에서 설치**:

   ```bash
   git clone https://github.com/you/spec-kit-my-ext
   specify extension add --dev spec-kit-my-ext/
   ```

### 방법 2: ZIP 아카이브

ZIP 아카이브를 만들어 GitHub Releases에 호스팅하세요.

```bash
zip -r spec-kit-my-ext-1.0.0.zip extension.yml commands/ scripts/ docs/
```

사용자는 다음으로 설치합니다.

```bash
specify extension add <extension-name> --from https://github.com/.../spec-kit-my-ext-1.0.0.zip
```

### 방법 3: 커뮤니티 참조 카탈로그

공개 탐색을 위해 커뮤니티 카탈로그에 제출하세요.

1. 확장의 **GitHub 릴리스를 만듭니다**
2. [Extension Submission](https://github.com/github/spec-kit/issues/new?template=extension_submission.yml) 템플릿으로 **이슈를 제출합니다**
3. **검토 후** 메인테이너가 카탈로그를 갱신하고 확장을 사용할 수 있게 됩니다.
   - 사용자는 `catalog.community.json`을 둘러보며 확장을 찾을 수 있습니다
   - 사용자는 릴리스 아카이브를 직접 설치하거나 자신의 신뢰 카탈로그에 추가하기
     전에 검증합니다
   - 사용자는 `--from <archive-url>`로, 또는 검증한 카탈로그에서 설치합니다

자세한 제출 지침은 [확장 게시 가이드](publishing.md)를 참고하세요.

---

## 모범 사례

### 이름 짓기 관례

- **확장 ID**: 설명적인 하이픈 이름을 사용하세요(`ji`가 아니라 `jira-integration`)
- **명령**: 동사-명사 패턴을 사용하세요(`create-issue`, `sync-status`)
- **설정 파일**: 확장 ID에 맞추세요(`jira-config.yml`)

### 문서

- **README.md**: 개요, 설치, 사용법
- **CHANGELOG.md**: 버전 이력
- **docs/**: 상세 가이드
- **명령 설명**: 명확하고 간결하게

### 버전 관리

- **SemVer를 따르세요**: `MAJOR.MINOR.PATCH`
- **MAJOR**: 호환성을 깨는 변경
- **MINOR**: 새 기능
- **PATCH**: 버그 수정
- **콘텐츠가 바뀔 때마다 버전을 올리세요**: `specify extension update`의 업데이트
  제안은 버전을 기준으로 하므로, 버전을 올리지 않고 배포한 콘텐츠 변경은 이미
  설치된 사본에 자동으로 전달되지 않습니다. 이 저장소의 번들된 확장에서는 CI가
  버전 올리기를 강제합니다(`extension-version-guard.yml`). `extensions/<id>/`
  아래의 파일을 변경하는 PR은 그 확장의 `extension.yml` 버전도 올리고
  `extensions/catalog.json`을 동기화된 상태로 유지해야 합니다.

### 보안

- **시크릿을 커밋하지 마세요**: 환경 변수를 사용하세요
- **입력을 검증하세요**: 사용자 인자를 정제하세요
- **권한을 문서화하세요**: 어떤 파일/API에 접근하는지

### 호환성

- **버전 범위를 지정하세요**: 정확한 버전을 요구하지 마세요
- **여러 버전으로 테스트하세요**: 호환성을 보장하세요
- **우아한 기능 저하**: 누락된 기능을 처리하세요

---

## 확장 예시

### 최소 확장

가능한 가장 작은 확장:

```yaml
# extension.yml
schema_version: "1.0"
extension:
  id: "minimal"
  name: "Minimal Extension"
  version: "1.0.0"
  description: "Minimal example"
requires:
  speckit_version: ">=0.1.0"
provides:
  commands:
    - name: "speckit.minimal.hello"
      file: "commands/hello.md"
```

````markdown
<!-- commands/hello.md -->
---
description: "Hello command"
---

# Hello World

```bash
echo "Hello, $ARGUMENTS!"
```
````

### 설정이 있는 확장

설정을 사용하는 확장:

```yaml
# extension.yml
# ... metadata ...
provides:
  config:
    - name: "tool-config.yml"
      template: "tool-config.template.yml"
      required: true
```

```yaml
# tool-config.template.yml
api_endpoint: "https://api.example.com"
timeout: 30
```

````markdown
<!-- commands/use-config.md -->
# Use Config

Load config:
```bash
config_file=".specify/extensions/tool/tool-config.yml"
endpoint=$(yq eval '.api_endpoint' "$config_file")
echo "Using endpoint: $endpoint"
```
````

### 훅이 있는 확장

자동으로 실행되는 확장:

```yaml
# extension.yml
hooks:
  after_tasks:
    command: "speckit.auto.analyze"
    optional: false  # Always run
    description: "Analyze tasks after generation"
```

한 이벤트에 여러 명령을 두면 현재 명령 템플릿은 YAML 순서로 드러냅니다(반면
`HookExecutor`는 `priority`로 정렬하며 낮은 것이 먼저).

```yaml
# extension.yml
hooks:
  after_plan:
    - command: "speckit.my-ext.verify"
      priority: 5
      optional: false
      description: "Verify the plan"
    - command: "speckit.my-ext.report"
      priority: 10
      optional: true
      prompt: "Generate the report?"
      description: "Generate a report from the plan"
```

---

## 문제 해결

### 확장이 설치되지 않음

**오류**: `Invalid extension ID`

- **해결**: 소문자, 영숫자, 하이픈만 사용하세요

**오류**: `Extension requires spec-kit >=0.2.0`

- **해결**: [업그레이드 가이드](../upgrade.md)에 따라 Spec Kit을 업그레이드하세요. `uv tool install specify-cli --force --from git+https://github.com/github/spec-kit.git`은 소스 설치 폴백으로 계속 사용할 수 있습니다. PyPI에서 설치했고 그 경로를 유지하고 싶다면 [PyPI 업그레이드 안내](../install/pypi.md#upgrade)를 따르세요.

**오류**: `Command file not found`

- **해결**: 매니페스트에 지정한 경로에 명령 파일이 존재하는지 확인하세요

### 스킬이 등록되지 않음

**증상**: 스킬이 GitHub Copilot 채팅에 나타나지 않음

**확인**:

1. 생성된 스킬이 `.github/skills/speckit-my-ext-hello/SKILL.md`에 존재하는지
2. 확장이 성공적으로 설치되었는지
3. 명령이 레지스트리에 등록되었는지:

   ```bash
   cat .specify/extensions/.registry
   ```

**해결**: Copilot이
[스킬 레이아웃](../integrations/copilot.md#default-skills-layout)으로 설정되어
있는지 확인한 다음, 확장을 다시 설치해 등록을 트리거하세요.

### 설정이 로드되지 않음

**확인**:

1. 설정 파일이 존재하는지: `.specify/extensions/{ext-id}/{ext-id}-config.yml`
2. YAML 구문이 유효한지: `yq eval '.' config.yml`
3. 환경 변수가 올바르게 설정되었는지

---

## 도움 받기

- **이슈**: GitHub 저장소에 버그를 보고하세요
- **토론**: GitHub Discussions에서 질문하세요
- **예시**: 번들된 [GitHub](https://github.com/github/spec-kit/tree/main/extensions/github)
  및 [bug](https://github.com/github/spec-kit/tree/main/extensions/bug) 확장을
  참고하세요

---

## 다음 단계

1. 이 가이드를 따라 **확장을 만드세요**
2. `--dev` 플래그로 **로컬에서 테스트하세요**
3. **커뮤니티와 공유하세요**(GitHub, 카탈로그)
4. 피드백에 따라 **반복 개선하세요**

즐거운 확장 개발 되세요! 🚀

> 이 페이지의 에이전트 호출 예시는 GitHub Copilot의 기본 스킬 구문(`/speckit-*`)을 사용합니다. 다른 통합과 모드는 [명령 호출](../integrations/cli.md#command-invocation)을 참고하세요.

## 함께 보기

- [확장 API 레퍼런스](api.md)
- [확장 CLI 레퍼런스](cli.md)
- [확장 카탈로그](catalogs.md)
- [확장 게시하기](publishing.md)
- [확장 사용자 가이드](overview.md)
