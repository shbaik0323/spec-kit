# 업그레이드 가이드

> Spec Kit을 이미 설치했고, 새 기능·버그 수정·갱신된 슬래시 명령을 받기 위해 최신 버전으로 업그레이드하려는 경우를 위한 문서입니다. 이 가이드는 CLI 도구 업그레이드와 프로젝트 파일 업데이트를 모두 다룹니다.

---

## 빠른 참조

| 업그레이드 대상 | 명령 | 사용 시점 |
|----------------|---------|-------------|
| **CLI 도구(권장)** | `specify self upgrade` | 최신 안정 릴리스로 제자리 업그레이드합니다. `uv tool`로 설치했는지 `pipx`로 설치했는지 자동 감지합니다. |
| **CLI 도구 — 버전 고정** | `specify self upgrade --tag vX.Y.Z[suffix]` | 최신 안정 버전 대신 특정 릴리스 태그로 업그레이드합니다. 접미사는 dev, alpha/beta/rc, 빌드 메타데이터 형식으로 제한됩니다. |
| **CLI 도구 — 수동 폴백** | `uv tool install specify-cli --force --from git+https://github.com/github/spec-kit.git@vX.Y.Z` | `specify self upgrade`를 사용할 수 없거나(오래된 설치) 명시적으로 제어하고 싶을 때 사용합니다. |
| **CLI 도구 — 수동 폴백(pipx)** | `pipx install --force git+https://github.com/github/spec-kit.git@vX.Y.Z` | 위와 같으며 pipx 설치용입니다. |
| **프로젝트 파일** | `specify integration upgrade <key>` 실행 후 `specify extension update` 실행 | 프로젝트에 설치된 통합 파일과 확장을 새로 고칩니다 |
| **둘 다** | CLI 업그레이드 후 프로젝트 업데이트 실행 | 메이저 버전 업데이트에 권장합니다 |

---

## 1부: CLI 도구 업그레이드

CLI 도구(`specify`)는 프로젝트 파일과 별개입니다. 최신 기능과 버그 수정을 받으려면 업그레이드하세요.

PyPI에서 설치했고 그 설치 경로를 유지하고 싶다면
[PyPI 업그레이드 지침](install/pypi.md#upgrade)을 따르세요.
`uv tool install specify-cli`를 다시 실행해도 기존 설치는 업그레이드되지 않습니다.

### 권장: `specify self upgrade`

CLI에는 일반적인 경우를 자동으로 처리하는 두 가지 자체 관리 명령이 포함되어 있습니다.

```bash
# 더 새로운 릴리스가 있는지 확인(읽기 전용 — 아무것도 수정하지 않음)
specify self check

# 실제로 업그레이드하지 않고 실행될 내용을 미리 보기
specify self upgrade --dry-run

# 최신 안정 릴리스로 제자리 업그레이드(uv tool과 pipx 설치를 자동 감지)
specify self upgrade

# 또는 특정 릴리스 태그로 고정(vX.Y.Z[suffix]를 원하는 태그로 교체)
specify self upgrade --tag vX.Y.Z[suffix]
```

인자 없는 `specify self upgrade`는 `pip install -U`나 `npm update` 같은 명령의 무프롬프트 동작과 마찬가지로 즉시 실행됩니다. CLI는 런타임을 `uv tool`, `pipx`, `uvx (ephemeral)`, 소스 체크아웃, 미지원 중 하나로 분류합니다. `uv tool`과 `pipx`만 자동으로 업그레이드됩니다. `uv tool` 설치의 경우 고정된 릴리스 태그가 동작하도록 내부적으로 `uv tool install specify-cli --force --from <git ref>`를 실행합니다. Windows에서는 `uv`가 `specify`가 실행 중인 도구 환경을 교체할 수 없으므로, `uv tool` 설치에서 `specify self upgrade`는 종료 후 여러분이 직접 실행할 명령을 출력하고 아무것도 건드리지 않은 채 0으로 종료합니다. 그 환경에서 실행 중인 다른 프로세스(예: `specify mcp` 서버)를 먼저 중지하세요. 그러지 않으면 uv가 도중에 실패할 수 있습니다. 나머지 경로는 경로별 안내를 출력하고 아무것도 건드리지 않은 채 0으로 종료합니다.

도구 감지는 현재 및 레거시 기본 설치 디렉터리뿐 아니라 `UV_TOOL_DIR`, `PIPX_HOME`, `XDG_DATA_HOME`으로 설정한 디렉터리도 인식합니다. 위치를 옮긴 설치를 업그레이드할 때는 설치 프로그램도 그 위치를 대상으로 삼도록 같은 디렉터리 환경 변수를 설정된 채로 유지하세요.

`PIPX_HOME`은 맨 앞의 `~`를 홈 디렉터리로 확장합니다. `UV_TOOL_DIR`은 이를 작업 디렉터리 기준 상대 경로의 일부로 문자 그대로 취급합니다. `XDG_DATA_HOME`은 절대 경로여야 하며, `~`로 시작하는 값을 포함한 상대 값은 무시됩니다.

고정 태그는 `vMAJOR.MINOR.PATCH`로 시작해야 합니다. 선택적 접미사는 `v1.0.0-rc1`, `v0.8.0.dev0`, `v0.8.0+build.42`, 또는 조합인 `v1.0.0-rc1+build.42`처럼 dev, alpha/beta/rc, 빌드 메타데이터 형식으로 제한됩니다. 브랜치 이름, 해시 ref, `latest`, `v` 없는 버전은 거부됩니다.

설치 프로그램 하위 프로세스가 실행될 수 있는 시간을 제한하려면 `SPECIFY_UPGRADE_TIMEOUT_SECS`를 설정하세요(기본값: 타임아웃 없음 — 필요하면 `Ctrl+C`로 중단). 이 내부 타임아웃이 발동하면 `specify self upgrade`는 124로 종료하고, 설정된 타임아웃과 수동 재시도 명령을 포함해 설치 프로그램 하위 프로세스를 기다리다 시간이 초과되었다고 보고합니다. 설치 프로그램의 실제 종료 코드 124는 `Upgrade failed. Installer exit code: 124.`와 함께 전파되므로, 스크립트는 종료 코드 124를 모호한 것으로 취급하고 두 경우를 구분해야 할 때는 메시지를 확인해야 합니다.

설치된 CLI가 `specify self upgrade`를 도입한 릴리스보다 오래되었다면 아래의 수동 대응 명령을 사용하세요. 이 명령들은 설치 명령을 명시적으로 제어하고 싶을 때도 유용합니다.

### `uv tool install`로 설치한 경우

특정 릴리스로 업그레이드합니다(최신 태그는 [Releases](https://github.com/github/spec-kit/releases)에서 확인).

```bash
uv tool install specify-cli --force --from git+https://github.com/github/spec-kit.git@vX.Y.Z
```

### 일회성 `uvx` 명령을 사용하는 경우

원하는 릴리스 태그를 지정합니다.

```bash
uvx --from git+https://github.com/github/spec-kit.git@vX.Y.Z specify init --here --integration copilot
```

`uvx`는 그 명령 하나를 위해 Spec Kit의 임시 사본을 실행합니다. `uv tool install`, `pipx`, 또는 다른 도구 관리자로 설치한 영구 `specify`를 업데이트하지 않습니다. 새 기능이 `uvx`로는 동작하는데 로컬 `specify`가 여전히 이전 버전을 보고한다면, 설치 방법에 맞는 명령으로 영구 CLI를 업그레이드하세요.

### `pipx`로 설치한 경우

특정 릴리스로 업그레이드합니다.

```bash
pipx install --force git+https://github.com/github/spec-kit.git@vX.Y.Z
```

### 업그레이드 확인

```bash
# CLI가 동작하는지 확인하고 설치된 도구를 표시
specify check

# 설치된 버전을 최신 GitHub 릴리스와 대조해 확인
specify self check
```

`specify check`는 주변 도구 환경을 보여 줍니다. `specify self check`는 읽기 전용이며, 지금 최신 릴리스를 쓰고 있는지(`Up to date: X.Y.Z`) 아니면 그 사이에 더 새로운 릴리스가 나왔는지 알려 줍니다.

---

## 2부: 프로젝트 파일 업데이트

Spec Kit이 새 기능(새 슬래시 명령, 갱신된 템플릿, 확장 변경 등)을 릴리스하면 프로젝트에 설치된 Spec Kit 파일을 새로 고쳐야 합니다.

### 무엇이 업데이트되나요?

기존 Spec Kit 프로젝트에서는 매니페스트를 인식하는 업그레이드 경로를 먼저 사용하세요.

- ✅ **통합 명령/스킬 파일** (`.claude/skills/`, `.github/skills/`, `.agents/skills/` 등)
- ✅ **관리되는 공유 스크립트와 템플릿** (`.specify/scripts/`, `.specify/templates/`) — 이전 관리 사본에서 변경되지 않은 경우
- ✅ **설치된 확장** — `specify extension update`를 실행할 때

통합 업그레이드 명령은 설치 매니페스트를 사용해 로컬 편집을 감지합니다. 관리되는 통합 파일이 설치 후 수정되었다면 명령이 멈추고, 변경 내용을 확인하거나 `--force`로 다시 실행하라고 요청합니다.

### 무엇이 안전하게 유지되나요?

다음 파일은 매니페스트 인식 통합/확장 업그레이드 경로가 **절대 건드리지 않습니다**.

- ✅ **여러분의 명세** (`specs/001-my-feature/spec.md` 등) - **안전 확인됨**
- ✅ **여러분의 구현 계획** (`specs/001-my-feature/plan.md`, `tasks.md` 등) - **안전 확인됨**
- ✅ **여러분의 컨스티튜션** (`.specify/memory/constitution.md`) — `specify integration upgrade` 사용 시
- ✅ **여러분의 소스 코드** - **안전 확인됨**
- ✅ **여러분의 git 이력** - **안전 확인됨**

`specs/` 디렉터리는 템플릿 패키지에서 완전히 제외되며 업그레이드 중에 절대 수정되지 않습니다.

### 1. 설치된 통합 확인

프로젝트 디렉터리 안에서 다음을 실행합니다.

```bash
specify integration status
```

기본 통합, 설치된 모든 통합, 수정되었거나 누락된 관리 파일을 보고합니다. `.specify/integration.json`을 직접 확인할 수도 있습니다. 설치된 통합은 `installed_integrations` 아래에 나열됩니다.

### 2. 설치된 각 통합 업그레이드

프로젝트 디렉터리 안에서 다음을 실행합니다.

```bash
specify integration upgrade <key>
```

`<key>`를 `copilot`, `claude`, `codex` 같은 설치된 통합 키로 바꾸세요. 여러 통합이 설치된 프로젝트에서는 설치된 키마다 한 번씩 명령을 실행합니다.

**예시:**

```bash
specify integration upgrade claude
specify integration upgrade codex
```

`--script`, `--integration-options`, `--force` 같은 옵션은 [통합 레퍼런스](integrations/cli.md#upgrade-an-integration)를 참고하세요.

### 3. 설치된 확장 업데이트

다음을 실행합니다.

```bash
specify extension update
```

확장 인자가 없으면 설치된 모든 확장을 업데이트합니다. 확장 하나만 업데이트하려면 `specify extension update <extension-id-or-name>`을 사용하세요. 자세한 내용은 [확장 CLI 레퍼런스](extensions/cli.md#update-extensions)를 참고하세요.

### 폴백: init 다시 실행

프로젝트가 매니페스트 도입 이전에 만들어졌거나, 통합 메타데이터가 누락되었거나, 더 광범위한 복구가 필요하다면 여전히 init을 다시 실행할 수 있습니다.

```bash
specify init --here --force --integration <your-agent>
```

이는 기본 프로젝트 파일 업그레이드 경로가 아니라 탈출구로 사용하세요. 선택한 통합과 공유 프로젝트 스캐폴딩을 새로 고치지만, 파일을 덮어쓰기 전에 통합별 매니페스트 검사를 동일하게 수행하지는 않습니다.

## ⚠️ 중요 경고

### 1. 컨스티튜션 파일과 메모리 사용자 정의

`specify integration upgrade <key>`는 `.specify/memory/constitution.md`를 업데이트하지 않습니다.

폴백인 `specify init --here --force --integration <your-agent>` 경로도 기존 `.specify/memory/constitution.md`를 보존합니다. 파일이 없으면 init이 현재 컨스티튜션 템플릿으로부터 생성합니다. 매니페스트 인식 업그레이드 경로에서는 컨스티튜션 백업/복원 단계가 필요하지 않습니다.

광범위한 폴백 새로 고침이 늘 그렇듯, `init --here --force`를 사용하기 전에 로컬 사용자 정의를 커밋하거나 백업해 결과 diff를 검토할 수 있게 하세요.

### 2. 통합, 스크립트, 템플릿의 사용자 수정

`specify integration upgrade <key>`는 매니페스트로 추적되는 통합 파일이 로컬에서 수정된 경우 `--force`를 전달하지 않는 한 차단됩니다.

공유 스크립트와 템플릿은 이전에 기록된 관리 사본과 여전히 일치할 때 새로 고쳐집니다. 로컬 사용자 정의는 덮어쓰는 force/refresh 옵션을 명시적으로 사용하지 않는 한 보존됩니다. `.specify/scripts/`나 `.specify/templates/`의 파일을 사용자 정의했다면 먼저 커밋하거나 백업하세요.

```bash
# 사용자 정의 템플릿과 스크립트 백업
cp -r .specify/templates .specify/templates-backup
cp -r .specify/scripts .specify/scripts-backup

# 업그레이드 후 변경 사항을 수동으로 다시 병합
```

### 3. 중복 슬래시 명령(IDE 기반 에이전트)

일부 IDE 기반 에이전트(Kilo Code, Cline 등)는 업그레이드 후 **중복 슬래시 명령**을 표시할 수 있습니다. 이전 버전과 새 버전이 모두 나타납니다.

**해결:** 에이전트 폴더에서 오래된 명령 파일을 수동으로 삭제하세요.

**Kilo Code 예시:**

```bash
# 현재 및 레거시 Kilo 명령 폴더 나열
ls -la .kilo/commands/
ls -la .kilocode/workflows/

# 이전 버전 삭제(예시 파일명 - 실제와 다를 수 있음)
rm .kilocode/workflows/speckit.specify-old.md
rm .kilocode/workflows/speckit.plan-v1.md
```

IDE를 다시 시작해 명령 목록을 새로 고치세요.

---

## 동작 변경: `/constitution`이 더 이상 템플릿으로 전파되지 않음

`/constitution` 명령([#3790](https://github.com/github/spec-kit/pull/3790))은 자신의
아티팩트로 범위가 한정됩니다. `.specify/memory/constitution.md`를 갱신하고 Sync Impact
Report를 작성하며, `plan-template.md`, `spec-template.md`, `tasks-template.md`, 설치된
명령 파일, 안내 문서를 **더 이상 편집하지 않습니다**.

### 이유

Spec Kit은 **런타임 해석**을 사용합니다. `plan`, `tasks`, `analyze`는 실행할 때마다
`.specify/memory/constitution.md`를 실시간으로 읽고, `analyze`가 전용 드리프트
검사기입니다. 관리 대상 템플릿은 사본이 아니라 포인터를 담습니다. `plan-template.md`는
`[Gates determined based on constitution file]`을 담은 채 제공되고, `/plan`은 실행할
때마다 살아 있는 컨스티튜션으로부터 그 섹션을 채웁니다. 전파는 단일 진실 공급원을
중복시켰고 프리셋/오버라이드 합성 시스템과 충돌했습니다(`replace` 프리셋이 편집된 코어
템플릿을 가립니다).

더 넓게 보면, Spec Kit은 이제 제자리 파일 편집이 아니라 프리셋과 확장으로 공유 자산을
관리합니다. 해석 스택을 통해 정책을 합성하면, 코어 팀이 볼 수 없는 저장소별 사본에
고정되는 대신 저장소 전반에서 중앙 소유, 버전 관리, 감사가 가능한 상태로 유지됩니다.

### 기존 프로젝트에 호환성을 깨는 변경인가요?

**아니요. 워크플로는 계속 동작합니다.** `/constitution`이 해당 파일을 제자리에서
편집하는 데 의존했을 때만 차이를 느낄 것입니다. 템플릿은 스캐폴드이지 권위 있는 원본이
아닙니다. `/plan`을 실행하면 템플릿을 기능별 `plan.md`로 복사하고 살아 있는
컨스티튜션으로부터 Constitution Check를 다시 도출하며, `/analyze`가 그에 비추어
검증합니다. 이전 `/constitution` 실행이 구체적인 게이트 텍스트를
`.specify/templates/plan-template.md`에 구체화해 두었더라도, 런타임에는 살아 있는
컨스티튜션이 여전히 진실 공급원입니다.

**강제하지 않은 업그레이드**에서는 구체화된 템플릿이 *보존*됩니다(해시가 기록된 관리
사본과 달라지므로 새로 고침이 이를 사용자 정의로 취급해 덮어쓰지 않습니다). 퇴행하는
것은 없습니다.

### 선택적 정리 — 런타임 포인터로 되돌리기

미리 채워져 고정된 Constitution Check는 다소 오해를 부르는 스캐폴드이며 첫 `/plan`
패스를 편향시킬 수 있습니다. 런타임 해석으로 완전히 돌아가려면
`.specify/templates/plan-template.md`의 섹션 본문을 포인터로 재설정하세요.

```text
## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

[Gates determined based on constitution file]
```

파일의 나머지는 그대로 두세요. 이는 정리 작업이지 필수 마이그레이션이 아닙니다.

### 이전 동작 유지(선택형)

팀이 구체화된 템플릿을 **검토되고 커밋된 아티팩트**로 취급하고 `/constitution`이 계속
전파하기를 원한다면, 번들로 제공되는 **`constitution-sync`** 프리셋을 설치하세요.

```bash
specify preset add constitution-sync
```

이 프리셋은 코어 `/constitution` 명령을 감싸 전파 패스를 다시 추가합니다. 버전이
지정된 프리셋·확장 제공 템플릿이나 명령 파일은 편집하지 **않습니다**(그것들은 각
패키지가 소유하며 조정 시 다시 합성됩니다). 이 제자리 편집 전파 모델은 나머지 SDD
명령이 프리셋/확장으로 관리될 때 사용하는 합성 모델과 충돌한다는 점에 유의하세요.
절충점과 기본 동작을 선호해야 하는 경우는 `presets/constitution-sync/README.md`의
"Interaction with the resolution stack" 섹션을 참고하세요.

---

## 흔한 시나리오

### 시나리오 1: "새 슬래시 명령만 받고 싶어요"

```bash
# CLI 업그레이드(uv tool과 pipx 설치를 자동 감지)
specify self upgrade

# 설치된 통합 확인
specify integration status

# 새 명령을 받도록 프로젝트 파일 업데이트
specify integration upgrade <key>
specify extension update
```

### 시나리오 2: "템플릿과 컨스티튜션을 사용자 정의했어요"

```bash
# 1. 사용자 정의를 커밋하거나 백업
git status
cp -r .specify/templates /tmp/templates-backup

# 2. CLI 업그레이드
specify self upgrade

# 3. 매니페스트 인식 프로젝트 업데이트를 먼저 사용
specify integration upgrade <key>
specify extension update

# 4. 업그레이드가 수정된 관리 파일을 보고하면 --force를 쓰기 전에 diff를 확인
```

### 시나리오 3: "IDE에 슬래시 명령이 중복으로 보여요"

IDE 기반 에이전트(Kilo Code, Cline 등)에서 발생합니다.

```bash
# Kilo Code의 경우 현재 및 레거시 명령 폴더를 모두 확인
ls -la .kilo/commands/
ls -la .kilocode/workflows/

# 오래된 명령 파일 삭제
rm .kilocode/workflows/speckit.old-command-name.md

# IDE 다시 시작
```

### 시나리오 4: "git 확장은 원하지 않아요"

git 확장은 이제 선택형이므로, 명시적으로 추가하지 않는 한 업그레이드가 이를 설치하지 않습니다.

```bash
# CLI 업그레이드
specify self upgrade

# 통합 파일과 설치된 확장 새로 고침
specify integration upgrade <key>
specify extension update

# `specify extension add git`을 실행하지 않는 한 git 확장은 추가되지 않음
```

나중에 git 확장의 명령과 훅이 필요해지면 명시적으로 설치하세요.

```bash
specify extension add git
```

Git을 사용하지 않는 프로젝트도 계획 명령 전에 `SPECIFY_FEATURE_DIRECTORY`를 기능 디렉터리 경로로 설정하면 Spec Kit을 사용할 수 있습니다.

```bash
# Bash/Zsh
export SPECIFY_FEATURE_DIRECTORY="specs/001-my-feature"

# PowerShell
$env:SPECIFY_FEATURE_DIRECTORY = "specs/001-my-feature"
```

또는 `.specify/feature.json`을 자동으로 생성하는 `/speckit.specify` 명령을 실행하세요.

---

## 문제 해결

### "업그레이드 후 슬래시 명령이 나타나지 않아요"

**원인:** 에이전트가 명령 파일을 다시 불러오지 않았습니다.

**해결:**

1. **IDE/에디터를 완전히 다시 시작**하세요(창 다시 불러오기만으로는 부족)
2. **CLI 기반 에이전트**는 파일이 존재하는지 확인하세요.

   ```bash
   ls -la .claude/skills/        # Claude Code
   ls -la .gemini/commands/      # Gemini
   ls -la .cursor/skills/      # Cursor
   ls -la .pi/prompts/           # Pi Coding Agent
   ls -la .omp/commands/         # Oh My Pi
   ```

3. **에이전트별 설정을 확인**하세요.
   - Codex는 `CODEX_HOME` 환경 변수가 필요합니다
   - 일부 에이전트는 워크스페이스 재시작이나 캐시 삭제가 필요합니다

### "init이 제 컨스티튜션 사용자 정의를 덮어쓰나요?"

현재의 `specify init --here --force`는 기존 `.specify/memory/constitution.md`를 보존합니다. 파일이 없을 때만 템플릿으로부터 생성합니다.

과거에 예전 워크플로나 수동 교체로 컨스티튜션 변경 사항을 잃었다면 git이나 백업에서 복원하세요.

```bash
# 사용자 정의한 컨스티튜션을 커밋해 둔 경우
git restore .specify/memory/constitution.md

# 수동으로 백업해 둔 경우
cp /tmp/constitution-backup.md .specify/memory/constitution.md
```

**예방:** 일상적인 프로젝트 파일 업데이트에는 `specify integration upgrade <key>`를 사용하세요. 폴백인 `specify init --here --force` 경로가 필요하다면 먼저 커밋해 이후 전체 diff를 검토할 수 있게 하세요.

### "Warning: Current directory is not empty"

**전체 경고 메시지:**

```text
Warning: Current directory is not empty (25 items)
Template files will be merged with existing content and may overwrite existing files
Do you want to continue? [y/N]
```

**의미:**

이 경고는 이미 파일이 있는 디렉터리에서 `specify init --here`(또는 `specify init .`)를 실행할 때 나타납니다. 다음을 알려 줍니다.

1. **디렉터리에 기존 내용이 있음** - 예시에서는 25개의 파일/폴더
2. **파일이 병합됨** - 새 템플릿 파일이 기존 파일 옆에 추가됩니다
3. **일부 파일이 덮어쓰일 수 있음** - 이미 Spec Kit 파일(`.claude/`, `.specify/` 등)이 있다면 새 버전으로 교체됩니다

**덮어쓰이는 것:**

Spec Kit 인프라 파일만 해당합니다.

- 에이전트 명령/스킬 파일(`.claude/skills/`, `.github/skills/` 등)
- `.specify/scripts/`의 스크립트
- `.specify/templates/`의 템플릿
- `.specify/memory/constitution.md` 같은 누락된 메모리 파일은 템플릿으로부터 생성될 수 있으며, 기존 컨스티튜션은 보존됩니다

**건드리지 않는 것:**

- `specs/` 디렉터리(명세, 계획, 태스크)
- 소스 코드 파일
- `.git/` 디렉터리와 git 이력
- Spec Kit 템플릿에 속하지 않는 그 밖의 파일

**대응 방법:**

- **`y`를 입력하고 Enter** - 폴백 init 경로를 사용할 때 병합을 진행합니다
- **`n`을 입력하고 Enter** - 작업을 취소합니다
- **`--force` 플래그 사용** - 이 확인을 완전히 건너뜁니다.

  ```bash
  specify init --here --force --integration copilot
  ```

**이 경고가 나타나는 경우:**

- ✅ 기존 Spec Kit 프로젝트에서 폴백 init 경로를 사용할 때는 **예상된 동작**입니다
- ✅ 기존 코드베이스에 Spec Kit을 추가할 때는 **예상된 동작**입니다
- ⚠️ 빈 디렉터리에 새 프로젝트를 만든다고 생각했다면 **예상 밖의 동작**입니다

**예방 팁:** 폴백 init 경로를 사용하기 전에 현재 작업을 커밋해, 새로 고쳐진 파일을 쉽게 검토하거나 복원할 수 있게 하세요.

### "CLI 업그레이드가 동작하지 않는 것 같아요"

명령이 이전 Spec Kit 버전처럼 동작한다면 먼저 CLI 자체에 물어보세요.

```bash
# 읽기 전용 — "Up to date: X.Y.Z" 또는 "Update available: X.Y.Z → vY.Z.W" 출력
specify self check

# 업그레이드가 사용할 설치 방법, 현재 버전, 대상 태그 미리 보기
specify self upgrade --dry-run
```

`specify check`는 오프라인 환경 스캔이고, `specify self check`는 CLI 버전 조회입니다.

`self check`가 잘못된 버전을 보여 주면 설치를 확인하세요.

```bash
# 설치된 도구 확인
uv tool list

# specify-cli가 표시되어야 함

# 경로 확인
which specify

# uv tool 설치 디렉터리를 가리켜야 함
```

찾을 수 없다면 다시 설치하세요.

```bash
uv tool uninstall specify-cli
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

### "프로젝트를 열 때마다 specify를 실행해야 하나요?"

**짧은 답:** 아니요. `specify init`은 프로젝트당 한 번만 실행하거나, 나중에 폴백 복구 경로로 실행합니다.

**설명:**

`specify` CLI 도구는 다음 용도로 사용합니다.

- **초기 설정:** 프로젝트에 Spec Kit을 부트스트랩하는 `specify init`
- **일상적인 프로젝트 파일 업그레이드:** `specify integration upgrade <key>`와 `specify extension update`
- **폴백 복구:** 통합 메타데이터가 누락되었거나 매니페스트 인식 경로를 쓸 수 없을 때의 `specify init --here --force`
- **진단:** 도구 설치를 확인하는 `specify check`

`specify init`을 한 번 실행하면 슬래시 명령(`/speckit.specify`, `/speckit.plan` 등)이 프로젝트의 에이전트 폴더(`.claude/`, `.github/skills/`, `.pi/prompts/`, `.omp/commands/` 등)에 **영구적으로 설치**됩니다. AI 코딩 에이전트가 이 명령 파일을 직접 읽으므로 `specify`를 다시 실행할 필요가 없습니다.

**에이전트가 슬래시 명령을 인식하지 못한다면:**

1. **명령 파일이 존재하는지 확인하세요.**

   ```bash
   # GitHub Copilot(기본 스킬 레이아웃)
   ls -la .github/skills/

   # 명령 레이아웃을 쓰는 GitHub Copilot(--integration-options="--commands",
   # 또는 스킬이 기본값이 되기 전에 만들어져 이후 업그레이드된 프로젝트)
   ls -la .github/agents/ .github/prompts/

   # Claude
   ls -la .claude/skills/

   # Pi
   ls -la .pi/prompts/

   # Oh My Pi
   ls -la .omp/commands/
   ```

2. **IDE/에디터를 완전히 다시 시작하세요**(창 다시 불러오기만으로는 부족)

3. `specify init`을 실행한 **올바른 디렉터리에 있는지 확인하세요**

4. **일부 에이전트**는 워크스페이스를 다시 불러오거나 캐시를 삭제해야 할 수 있습니다

**관련 문제:** Copilot이 로컬 파일을 열지 못하거나 예기치 않게 PowerShell 명령을 사용한다면, 이는 대개 IDE 컨텍스트 문제이며 `specify`와는 관련이 없습니다. 다음을 시도하세요.

- VS Code 다시 시작
- 파일 권한 확인
- 워크스페이스 폴더가 제대로 열려 있는지 확인

---

## 버전 호환성

Spec Kit은 메이저 릴리스에 시맨틱 버저닝을 따릅니다. CLI와 프로젝트 파일은 같은 메이저 버전 안에서 호환되도록 설계되었습니다.

**모범 사례:** 메이저 버전이 바뀔 때는 CLI와 프로젝트 파일을 함께 업그레이드해 동기화된 상태로 유지하세요.

---

## 다음 단계

업그레이드 후:

- **새 슬래시 명령 테스트:** `/speckit.constitution`이나 다른 명령을 실행해 모든 것이 동작하는지 확인하세요
- **릴리스 노트 검토:** [GitHub Releases](https://github.com/github/spec-kit/releases)에서 새 기능과 호환성을 깨는 변경을 확인하세요
- **워크플로 갱신:** 새 명령이 추가되었다면 팀의 개발 워크플로를 갱신하세요
- **문서 확인:** 갱신된 가이드는 [github.io/spec-kit](https://github.github.io/spec-kit/)에서 확인하세요
