# 확장 API 레퍼런스

Spec Kit 확장 시스템 API와 매니페스트 스키마에 대한 기술 레퍼런스입니다.

## 목차

1. [확장 매니페스트](#extension-manifest)
2. [Python API](#python-api)
3. [명령 파일 형식](#command-file-format)
4. [설정 스키마](#configuration-schema)
5. [훅 시스템](#hook-system)
6. [확장 런타임 이벤트](#extension-runtime-events)

---

<a id="extension-manifest"></a>

## 확장 매니페스트

### 스키마 버전 1.0

파일: `extension.yml`

```yaml
schema_version: "1.0"  # Required

extension:
  id: string           # Required, pattern: ^[a-z0-9-]+$
  name: string         # Required, human-readable name
  version: string      # Required, PEP 440 version; SemVer recommended
  description: string  # Required, brief description
  author: string       # Optional
  repository: string   # Optional, source URL
  license: string      # Optional (e.g., "MIT", "Apache-2.0")
  homepage: string     # Optional, homepage URL

requires:
  speckit_version: string  # Required, version specifier (>=X.Y.Z)
  tools:                   # Optional, array of tool requirements
    - name: string         # Tool name
      version: string      # Optional, version specifier
      required: boolean    # Optional, default: false

provides:
  commands:              # At least one of commands/templates/scripts/hooks/events required
    - name: string       # Required, pattern: ^speckit\.[a-z0-9-]+\.[a-z0-9-]+$
      file: string       # Required, relative path to command file
      description: string # Optional

  templates:             # Optional, array of declared templates. Always resolve
                          # as "replace" -- 'strategy' is not an authorable field here.
    - name: string       # Required, pattern: ^[a-z0-9-]+$
      file: string       # Required, relative path to template file
      description: string # Optional

  scripts:               # Optional, array of declared scripts. Always resolve
                          # as "replace" -- 'strategy' is not an authorable field here.
    - name: string       # Required, pattern: ^[a-z0-9-]+$
      file: string       # Required, relative path to script file
      description: string # Optional
      runtimes: [string] # Optional, subset of: bash, powershell, python

  config:                # Optional, array of config files
    - name: string       # Config file name
      template: string   # Template file path
      description: string
      required: boolean  # Default: false

hooks:                   # Optional, event hooks. Each event accepts either form below.
  event_name:            # e.g., "after_specify", "after_plan", "after_tasks", "after_implement"
    command: string      # Command to execute
    priority: integer    # Optional, >= 1, default 10 (HookExecutor sort metadata)
    optional: boolean    # Default: true
    prompt: string       # Prompt text for optional hooks
    description: string  # Hook description
    condition: string    # Optional, condition expression
  another_event:         # Any event may instead use a list of mappings (multiple commands)
    - command: string    # Same fields as the single mapping, per entry
      priority: integer
    - command: string
      priority: integer

events:                  # Optional, coding-agent lifecycle events
  session_start:
    command: string      # Command template with a runnable scripts entry

tags:                    # Optional, array of tags (2-10 recommended)
  - string

defaults:                # Optional, default configuration values
  key: value             # Any YAML structure

config:                  # Optional, separate from provides.config
  defaults:              # Merged over top-level defaults
    key: value
```

### 필드 명세

#### `extension.id`

- **타입**: string
- **패턴**: `^[a-z0-9-]+$`
- **설명**: 고유한 확장 식별자
- **예시**: `jira`, `linear`, `azure-devops`
- **유효하지 않음**: `Jira`, `my_extension`, `extension.id`

#### `extension.version`

- **타입**: string
- **형식**: PEP 440. 릴리스에는 `MAJOR.MINOR.PATCH`를 권장
- **설명**: 확장 버전
- **예시**: `1.0.0`, `0.9.5`, `2.1.3`
- **받아들여지는 것**: `v1.0`, `1.0`, `1.0.0-beta`
- **유효하지 않음**: `latest`, `not-a-version`

#### `requires.speckit_version`

- **타입**: string
- **형식**: 버전 지정자
- **설명**: 필요한 spec-kit 버전 범위
- **예시**:
  - `>=0.1.0` - 0.1.0 이상의 모든 버전
  - `>=0.1.0,<2.0.0` - 버전 0.1.x 또는 1.x
  - `==0.1.0` - 정확히 0.1.0
- **받아들여지는 것**: `>= 0.1.0`(연산자 뒤의 공백)
- **유효하지 않음**: `0.1.0`, `latest`

매니페스트 로딩은 여기에 비어 있지 않은 문자열을 요구합니다. 호환성 검사는 이를
`packaging.specifiers.SpecifierSet`으로 파싱해 설치된 Spec Kit 버전을 확인합니다.

#### `provides.commands[].name`

- **타입**: string
- **패턴**: `^speckit\.[a-z0-9-]+\.[a-z0-9-]+$`
- **설명**: 네임스페이스가 붙은 명령 이름
- **형식**: `speckit.{extension-id}.{command-name}`
- **예시**: `speckit.jira.specstoissues`, `speckit.linear.sync`
- **유효하지 않음**: `another-extension.command`, `speckit.jira.CreateIssues`

레거시 두 세그먼트 이름인 `speckit.command`와 `<extension-id>.command`는 경고와
함께 `speckit.<extension-id>.command`로 정규화됩니다. 작성자는 이 호환성 동작에
의존하지 말고 정식 이름을 사용해야 합니다.
설치는 기본 이름이 확장의 네임스페이스에 속하는지도 확인하며, 설치된 확장과의
정확한 명령 이름 또는 별칭 충돌, 그리고 코어 명령과의 정규화된 출력 이름 충돌을
거부합니다.

#### `provides.templates[].name` / `provides.scripts[].name`

- **타입**: string
- **패턴**: `^[a-z0-9-]+$`
- **설명**: 명령과 달리 템플릿과 스크립트는 이름으로 호출되지 않으므로,
  네임스페이스 명령 패턴이 아니라 `extension.id`와 같은 단순한 슬러그 패턴을
  사용합니다.
- **예시**: `myext-template`, `myext-collect`

#### `provides.templates[].strategy` / `provides.scripts[].strategy`

- 작성할 수 있는 필드가 아닙니다. 확장이 기여한 템플릿과 스크립트는 항상
  `replace`로 해석됩니다. 이런 항목에 `strategy` 키를 포함한 매니페스트는
  `ValidationError`로 거부됩니다. 합성 가능한 전략(`wrap`/`prepend`/`append`)은
  프리셋 전용입니다.

#### `provides.scripts[].runtimes`

- **타입**: 문자열 배열
- **값**: `bash`, `powershell`, `python`
- **설명**: 스크립트가 어떤 런타임을 지원하는지 선언합니다. 순수하게 참고용
  메타데이터이며 스크립트를 선택하거나 호출하는 데 쓰이지 않습니다.

#### `hooks`

- **타입**: object
- **키**: 이벤트 이름(예: `after_specify`, `after_plan`, `after_tasks`, `after_implement`, `before_analyze`)
- **값**: 단일 훅 매핑, 또는 한 이벤트에 여러 명령을 등록하는 훅 매핑 목록
- **설명**: 라이프사이클 이벤트에서 실행되는 훅
- **이벤트**: 코어 spec-kit 명령이 정의
- **순서**: 현재 명령 템플릿은 설정된 YAML 순서로 훅을 드러냅니다.
  반면 `HookExecutor.get_hooks_for_event()`는 `priority`(1 이상의 정수, 기본값 10)
  오름차순으로 반환하며, 동률은 안정적인 작성 순서를 따릅니다. 실행 제한 사항은
  [훅 설정](#hook-configuration)을 참고하세요.

#### `events`

- **타입**: object
- **키**: 정식 코딩 에이전트 이벤트 이름
- **값**: 실행 가능한 명령을 참조하는 이벤트 매핑
- **설명**: 설치된 코딩 에이전트의 네이티브 이벤트 설정으로 변환되는, 통합에
  중립적인 선언.
  [확장 런타임 이벤트](#extension-runtime-events)를 참고하세요.

---

<a id="python-api"></a>

## Python API

### ExtensionManifest

**모듈**: `specify_cli.extensions`

```python
from specify_cli.extensions import ExtensionManifest

manifest = ExtensionManifest(Path("extension.yml"))
```

**속성**:

```python
manifest.id                        # str: Extension ID
manifest.name                      # str: Extension name
manifest.version                   # str: Version
manifest.description               # str: Description
manifest.requires_speckit_version  # str: Required spec-kit version
manifest.commands                  # List[Dict]: Command definitions
manifest.templates                 # List[Dict]: Declared template definitions
manifest.scripts                   # List[Dict]: Declared script definitions
manifest.hooks                     # Dict: Hook definitions
```

**메서드**:

```python
manifest.get_hash()  # str: SHA256 hash of manifest file
```

**예외**:

```python
ValidationError       # Invalid manifest structure
CompatibilityError    # Incompatible with current spec-kit version
```

### ExtensionRegistry

**모듈**: `specify_cli.extensions`

```python
from specify_cli.extensions import ExtensionRegistry

registry = ExtensionRegistry(extensions_dir)
```

**메서드**:

```python
# Add extension to registry
registry.add(extension_id: str, metadata: dict)

# Remove extension from registry
registry.remove(extension_id: str)

# Get extension metadata
metadata = registry.get(extension_id: str)  # Optional[dict]

# List all extensions
extensions = registry.list()  # Dict[str, dict]

# Check if installed
is_installed = registry.is_installed(extension_id: str)  # bool
```

**레지스트리 형식**:

```json
{
  "schema_version": "1.0",
  "extensions": {
    "jira": {
      "version": "1.0.0",
      "source": {"kind": "catalog", "catalog": "my-org"},
      "manifest_hash": "sha256...",
      "enabled": true,
      "priority": 10,
      "registered_commands": {
        "claude": ["speckit.jira.specstoissues"]
      },
      "registered_skills": [],
      "installed_at": "2026-01-28T00:00:00"
    }
  }
}
```

`registered_commands`는 통합 키를 등록된 명령 이름에 매핑하고,
`registered_skills`는 보조 스킬 이름을 기록합니다. 설치 프로그램은 원시 로컬
출처로 `"local"`을, 또는 위와 같은 카탈로그 객체를 저장합니다.
CLI의 정규화된 `--json` 출력은 별도의
[공개 source 형태](cli.md#list-installed-extensions)를 사용합니다.

### ExtensionManager

**모듈**: `specify_cli.extensions`

```python
from specify_cli.extensions import ExtensionManager

manager = ExtensionManager(project_root)
```

**메서드**:

```python
# Install from directory
manifest = manager.install_from_directory(
    source_dir: Path,
    speckit_version: str,
    register_commands: bool = True
)  # Returns: ExtensionManifest

# Install from ZIP
manifest = manager.install_from_zip(
    zip_path: Path,
    speckit_version: str
)  # Returns: ExtensionManifest

# Remove extension
success = manager.remove(
    extension_id: str,
    keep_config: bool = False
)  # Returns: bool

# List installed extensions
extensions = manager.list_installed()  # List[Dict]

# Get extension manifest
manifest = manager.get_extension(extension_id: str)  # Optional[ExtensionManifest]

# Check compatibility
manager.check_compatibility(
    manifest: ExtensionManifest,
    speckit_version: str
)  # Raises: CompatibilityError if incompatible
```

### CatalogEntry

**모듈**: `specify_cli.extensions`

활성 카탈로그 스택의 카탈로그 하나를 나타냅니다.

```python
from specify_cli.extensions import CatalogEntry

entry = CatalogEntry(
    url="https://example.com/catalog.json",
    name="default",
    priority=1,
    install_allowed=True,
    description="Built-in catalog of installable extensions",
)
```

**필드**:

| 필드 | 타입 | 설명 |
|-------|------|-------------|
| `url` | `str` | 카탈로그 URL(localhost를 포함해 HTTPS를 사용해야 함) |
| `name` | `str` | 사람이 읽을 수 있는 카탈로그 이름 |
| `priority` | `int` | 정렬 순서(낮을수록 우선순위가 높고 충돌에서 이김) |
| `install_allowed` | `bool` | 이 카탈로그의 확장을 설치할 수 있는지 여부 |
| `description` | `str` | 카탈로그에 대한 선택적인 설명(기본값: 빈 문자열) |

### ExtensionCatalog

**모듈**: `specify_cli.extensions`

```python
from specify_cli.extensions import ExtensionCatalog

catalog = ExtensionCatalog(project_root)
```

**클래스 속성**:

```python
ExtensionCatalog.DEFAULT_CATALOG_URL    # default catalog URL
ExtensionCatalog.COMMUNITY_CATALOG_URL  # community catalog URL
```

**메서드**:

```python
# Get the ordered list of active catalogs
entries = catalog.get_active_catalogs()  # List[CatalogEntry]

# Fetch catalog (primary catalog, backward compat)
catalog_data = catalog.fetch_catalog(force_refresh: bool = False)  # Dict

# Search extensions across all active catalogs
# Each result includes _catalog_name and _install_allowed
results = catalog.search(
    query: Optional[str] = None,
    tag: Optional[str] = None,
    author: Optional[str] = None,
    verified_only: bool = False
)  # Returns: List[Dict]  — each dict includes _catalog_name, _install_allowed

# Get extension info (searches all active catalogs)
# Returns None if not found; includes _catalog_name and _install_allowed
ext_info = catalog.get_extension_info(extension_id: str)  # Optional[Dict]

# Check cache validity (primary catalog)
is_valid = catalog.is_cache_valid()  # bool

# Clear all catalog caches
catalog.clear_cache()
```

**결과 주석 필드**:

`search()`와 `get_extension_info()`가 반환하는 각 확장 딕셔너리에는 다음이
포함됩니다.

| 필드 | 타입 | 설명 |
|-------|------|-------------|
| `_catalog_name` | `str` | 소스 카탈로그의 이름 |
| `_install_allowed` | `bool` | 이 카탈로그로부터의 설치가 허용되는지 여부 |

**카탈로그 설정 파일** (`.specify/extension-catalogs.yml`):

```yaml
catalogs:
  - name: "default"
    url: "https://raw.githubusercontent.com/github/spec-kit/main/extensions/catalog.json"
    priority: 1
    install_allowed: true
    description: "Built-in catalog of installable extensions"
  - name: "community"
    url: "https://raw.githubusercontent.com/github/spec-kit/main/extensions/catalog.community.json"
    priority: 2
    install_allowed: false
    description: "Community-contributed extensions (discovery only)"
```

### HookExecutor

**모듈**: `specify_cli.extensions`

```python
from specify_cli.extensions import HookExecutor

hook_executor = HookExecutor(project_root)
```

**메서드**:

```python
# Get project config
config = hook_executor.get_project_config()  # Dict

# Save project config
hook_executor.save_project_config(config: Dict)

# Register hooks
hook_executor.register_hooks(manifest: ExtensionManifest)

# Unregister hooks
hook_executor.unregister_hooks(extension_id: str)

# Get hooks for event
hooks = hook_executor.get_hooks_for_event(event_name: str)  # List[Dict]

# Check if hook should execute
should_run = hook_executor.should_execute_hook(hook: Dict)  # bool

# Format hook message
message = hook_executor.format_hook_message(
    event_name: str,
    hooks: List[Dict]
)  # str
```

### CommandRegistrar

**모듈**: `specify_cli.extensions`

```python
from specify_cli.extensions import CommandRegistrar

registrar = CommandRegistrar()
```

**메서드**:

```python
# Register commands for Claude Code
registered = registrar.register_commands_for_claude(
    manifest: ExtensionManifest,
    extension_dir: Path,
    project_root: Path
)  # Returns: List[str] (command names)

# Parse frontmatter
frontmatter, body = registrar.parse_frontmatter(content: str)

# Render frontmatter
yaml_text = registrar.render_frontmatter(frontmatter: Dict)  # str
```

---

<a id="command-file-format"></a>

## 명령 파일 형식

### 범용 명령 형식

**파일**: `commands/{command-name}.md`

```markdown
---
description: "Command description"
tools:
  - 'mcp-server/tool_name'
  - 'other-mcp-server/other_tool'
---

# Command Title

Command documentation in Markdown.

## Prerequisites

1. Requirement 1
2. Requirement 2

## User Input

$ARGUMENTS

## Steps

### Step 1: Description

Instruction text...

\`\`\`bash
# Shell commands
\`\`\`

### Step 2: Another Step

More instructions...

## Configuration Reference

Information about configuration options.

## Notes

Additional notes and tips.
```

### 프런트매터 필드

```yaml
description: string   # Recommended, brief command description
tools: [string]       # Optional, MCP tools required
```

### 특수 변수

- `$ARGUMENTS` - 사용자가 제공한 인자의 자리 표시자
- Markdown 명령 출력에는 확장 컨텍스트 주석이 포함됩니다.

  ```markdown
  <!-- Extension: {extension-id} -->
  <!-- Config: .specify/extensions/{extension-id}/ -->
  ```

반면 생성된 스킬은 프런트매터의 `metadata.source`에 출처를 담으며 이 HTML 주석을
자동으로 포함하지 않습니다.
다른 출력 형식은 통합별 렌더러를 사용합니다.

---

<a id="configuration-schema"></a>

## 설정 스키마

### 확장 설정 파일

**파일**: `.specify/extensions/{extension-id}/{extension-id}-config.yml`

확장은 자체 설정 스키마를 정의합니다. 흔한 패턴은 다음과 같습니다.

```yaml
# Connection settings
connection:
  url: string
  api_key: string

# Project settings
project:
  key: string
  workspace: string

# Feature flags
features:
  enabled: boolean
  auto_sync: boolean

# Defaults
defaults:
  labels: [string]
  assignee: string

# Custom fields
field_mappings:
  internal_name: "external_field_id"
```

### 설정 레이어

1. **확장 기본값**(최상위 `defaults`가 `config.defaults`와 병합되며, 충돌 시
   `config.defaults`가 이김)
2. **프로젝트 설정** (`{extension-id}-config.yml`)
3. **로컬 오버라이드** (`local-config.yml`, gitignore 처리됨)
4. **환경 변수** (`SPECKIT_{EXTENSION}_*`)

`ConfigManager`는 호출될 때 이 병합을 수행합니다. 확장의 명령과 스크립트는 이를
사용하거나 자체 로딩을 구현해야 합니다. YAML의 `${...}` 문자열을 보간하거나
`config_schema` 필드를 검증하지 않습니다. 설정 검증은 확장의 명령이나 스크립트가
맡습니다.

### 환경 변수 패턴

형식: `SPECKIT_{EXTENSION}_{KEY}`

확장 ID는 하이픈이 밑줄로 바뀐 채 대문자가 됩니다. 그 접두사 뒤에서 밑줄로 구분된
각 구성 요소는 중첩된 소문자 키가 됩니다. `SPECKIT_JIRA_PROJECT_KEY`는
`project_key`가 아니라 `project.key`를 설정합니다.
환경 값은 문자열로 남습니다. 설치된 확장 ID들의 정규화된 접두사가 겹치면, 더 길게
일치하는 확장 ID가 그 변수를 소유합니다.

예시:

- `SPECKIT_JIRA_PROJECT_KEY`
- `SPECKIT_LINEAR_API_KEY`
- `SPECKIT_GITHUB_TOKEN`

---

<a id="hook-system"></a>

## 훅 시스템

### 훅 정의

각 이벤트는 단일 훅 매핑 또는 매핑 목록을 받습니다. 목록은 같은 이벤트에 여러 명령을 등록합니다.

**단일 매핑(extension.yml에서)**:

```yaml
hooks:
  after_tasks:
    command: "speckit.jira.specstoissues"
    optional: true
    prompt: "Create Jira issues from tasks?"
    description: "Automatically create Jira hierarchy"
    condition: null
```

**우선순위가 있는 매핑 목록**:

```yaml
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

하나의 매니페스트 목록 안에서 반복된 `command`는 "마지막이 이김" 방식으로 중복 제거되어 끝으로 옮겨지므로, 우선순위가 같은 경우의 동률도 작성 순서대로 정해집니다.

### 훅 이벤트

표준 이벤트(코어가 정의):

- `before_specify` - 명세 생성 전
- `after_specify` - 명세 생성 후
- `before_plan` - 구현 계획 전
- `after_plan` - 구현 계획 후
- `before_tasks` - 태스크 생성 전
- `after_tasks` - 태스크 생성 후
- `before_implement` - 구현 전
- `after_implement` - 구현 후
- `before_analyze` - 아티팩트 간 분석 전
- `after_analyze` - 아티팩트 간 분석 후
- `before_checklist` - 체크리스트 생성 전
- `after_checklist` - 체크리스트 생성 후
- `before_clarify` - 스펙 명확화 전
- `after_clarify` - 스펙 명확화 후
- `before_constitution` - 컨스티튜션 갱신 전
- `after_constitution` - 컨스티튜션 갱신 후

#### 번들된 `bug` 확장 이벤트(확장이 설치된 경우 사용 가능)

훅은 같은 세션에서 실행되는 프롬프트입니다. 대화에서 해석된 `BUG_SLUG`와 `BUG_DIR`(그리고 사후 훅의 경우 작성된 보고서)을 재사용하며, 변수는 주입되지 않습니다. 사전 훅은 slug 해석과 선행 조건 확인 후에 실행되고, 사후 훅은 보고서가 작성된 후 완료가 보고되기 전에 실행됩니다.

- `before_bug_assess` - 버그 디렉터리가 존재한 뒤, 보고를 받아들이기 전
- `after_bug_assess` - `assessment.md` 작성 후. 알아볼 수 없는 보고로 조기 중단되면 도달하지 않음
- `before_bug_fix` - 교정 방안을 적용하기 전
- `after_bug_fix` - `fix.md` 작성 후
- `before_bug_test` - 검증을 실행하기 전
- `after_bug_test` - `test.md` 작성 후

<a id="hook-configuration"></a>

### 훅 설정

Spec Kit은 프로젝트 수준 확장 등록과 훅 설정을 `.specify/extensions.yml`에
저장합니다. `installed` 목록은 설치된 확장 ID를 기록하고, `settings`는 프로젝트
전체 확장 설정을 저장하며, `hooks`는 등록된 훅을 이벤트별로 묶습니다.

```yaml
installed:
  - git
  - my-extension

settings:
  auto_execute_hooks: true

hooks:
  before_implement:
    - extension: git
      command: speckit.git.commit
      enabled: true
      optional: true
      priority: 10
      prompt: "Commit outstanding changes before implementation?"
      description: "Auto-commit before implementation"
      condition: null

  after_implement:
    - extension: my-extension
      command: speckit.my-extension.verify
      enabled: true
      optional: false
      priority: 5
      description: "Run verification after implementation"
```

`auto_execute_hooks`는 기본값이 `true`이지만 예약되어 있으며, 훅이 드러나거나
호출될 때 참조되지 않습니다.

| 필드 | 설명 |
| --- | --- |
| `extension` | 훅을 등록한 확장의 ID. |
| `command` | 훅에 연결된 확장 명령. |
| `enabled` | 훅이 활성인지 여부. `enabled: false`인 훅은 건너뜁니다. |
| `optional` | `true`이면 훅이 `prompt`와 함께 제시되며 건너뛸 수 있습니다. `false`이면 `EXECUTE_COMMAND` 마커와 함께 자동 훅으로 내보내집니다. |
| `priority` | 1 이상의 정수 메타데이터, 기본값 `10`. 현재 명령 템플릿은 우선순위 정렬이 아니라 설정된 YAML 순서를 사용합니다. |
| `prompt` | 선택적 훅을 실행할지 물을 때 표시되는 메시지. |
| `description` | 훅에 대한 사람이 읽을 수 있는 설명. |
| `condition` | `HookExecutor`가 평가하는 선택적 표현식: `is set`, `==`, `!=`와 함께 쓰는 `config.<path>` 또는 `env.<VAR>`. 현재 명령 템플릿은 조건을 평가하지 않고 비어 있지 않은 조건을 가진 훅을 건너뜁니다. |

훅 이벤트 이름은 훅이 언제 호출되는지를 식별하며, 일반적으로
`before_implement`, `after_implement`, `before_tasks`, `after_tasks`처럼
`before_<command>` 또는 `after_<command>`를 사용합니다.

확장 매니페스트는 설치 중에 유효하지 않은 훅 우선순위를 거부합니다.
기존 등록에 대해 `HookExecutor.get_hooks_for_event()`는 `normalize_priority()`를
사용해 정렬합니다. 누락된 값, 불리언, `int()`가 거부하는 값(유한하지 않은 부동
소수점 포함), `1`보다 작은 값은 `10`으로 폴백합니다.
숫자 문자열과 유한한 부동 소수점은 `int()`로 변환됩니다.
낮은 값이 먼저 반환되며, 동률은 안정적인 작성 순서를 따릅니다.
현재 명령 템플릿은 그 Python API를 사용하는 대신 훅 목록을 직접 읽으므로, 설정된
YAML 순서로 훅을 드러냅니다.

### 훅 메시지 형식

```markdown
## Extension Hooks

**Optional Hook**: {extension}
Command: `/{command}`
Description: {description}

Prompt: {prompt}
To execute: `/{command}`
```

또는 필수 훅의 경우:

```markdown
**Automatic Hook**: {extension}
Executing: `/{command}`
EXECUTE_COMMAND: {command}
```

---

<a id="extension-runtime-events"></a>

## 확장 런타임 이벤트

확장은 `extension.yml`에 최상위 `events` 매핑을 선언해 코딩 에이전트
라이프사이클 이벤트에 반응할 수 있습니다.

```yaml
events:
  session_start:
    command: speckit.my-extension.initialize
  pre_tool_use:
    command: speckit.my-extension.validate
```

이벤트 선언은 통합에 중립적입니다. Spec Kit은 이벤트를 지원하는 설치된 모든
통합을 새로 고치고, 각 통합은 정식 이벤트 이름을 코딩 에이전트의 네이티브 이벤트
설정으로 변환합니다. 지원되는 정식 이벤트는 `session_start`, `pre_tool_use`,
`post_tool_use`, `user_prompt_submit`, `stop`, `session_end`이며, 통합은 그 일부만
지원할 수 있습니다.

런타임 이벤트는 확장 훅과 별개입니다. 이벤트는 코딩 에이전트의 라이프사이클에
반응하며, 참조된 명령 템플릿의 `scripts` 항목을 직접 실행합니다. 따라서 참조된
명령은 실행 가능한 스크립트를 제공해야 합니다. 그 Markdown 지침은 코딩 에이전트에
전송되지 않습니다. 반면 훅은 `plan`, `tasks`, `implement` 같은 Spec Kit 명령
전후에 확장 명령을 드러냅니다.

---

## 예외

### ValidationError

확장 매니페스트 검증이 실패할 때 발생합니다.

```python
from specify_cli.extensions import ValidationError

try:
    manifest = ExtensionManifest(path)
except ValidationError as e:
    print(f"Invalid manifest: {e}")
```

### CompatibilityError

확장이 현재 spec-kit 버전과 호환되지 않을 때 발생합니다.

```python
from specify_cli.extensions import CompatibilityError

try:
    manager.check_compatibility(manifest, "0.1.0")
except CompatibilityError as e:
    print(f"Incompatible: {e}")
```

### ExtensionError

모든 확장 관련 오류의 기반 예외입니다.

```python
from specify_cli.extensions import ExtensionError

try:
    manager.install_from_directory(path, "0.1.0")
except ExtensionError as e:
    print(f"Extension error: {e}")
```

---

## 버전 함수

### version_satisfies

버전이 지정자를 충족하는지 확인합니다.

```python
from specify_cli.extensions import version_satisfies

# True if 1.2.3 satisfies >=1.0.0,<2.0.0
satisfied = version_satisfies("1.2.3", ">=1.0.0,<2.0.0")  # bool
```

---

## 파일 시스템 구조

```text
.specify/
├── extensions/
│   ├── .registry               # Extension registry (JSON)
│   ├── .cache/                 # Catalog cache
│   │   ├── catalog.json
│   │   └── catalog-metadata.json
│   ├── .backup/                # Config backups
│   │   └── {ext}-{config}.yml
│   ├── {extension-id}/         # Extension directory
│   │   ├── extension.yml       # Manifest
│   │   ├── {ext}-config.yml    # User config
│   │   ├── local-config.yml    # Local overrides (gitignored)
│   │   ├── {ext}-config.template.yml  # Template
│   │   ├── commands/           # Command files
│   │   │   └── *.md
│   │   ├── scripts/            # Helper scripts
│   │   │   └── *.sh
│   │   ├── docs/               # Documentation
│   │   └── README.md
├── extensions.yml              # Project extension config
└── scripts/                    # (existing spec-kit)

.claude/
└── commands/
    └── speckit.{ext}.{cmd}.md  # Registered commands
```

---

*최종 업데이트: 2026-01-28*
*API 버전: 1.0*
*Spec Kit 버전: 0.1.0*

## 함께 보기

- [확장 개발하기](developing.md)
- [확장 CLI 레퍼런스](cli.md)
- [확장 카탈로그](catalogs.md)
- [확장 사용자 가이드](overview.md)
- [확장 게시하기](publishing.md)
