# 핵심 명령

핵심 `specify` 명령은 프로젝트 초기화, 시스템 점검, 버전 정보를 다룹니다.

## 프로젝트 초기화

```bash
specify init [<project_name>]
```

| 옵션                     | 설명                                                                     |
| ------------------------ | ------------------------------------------------------------------------ |
| `--integration <key>`    | 사용할 AI 코딩 에이전트 통합(예: `copilot`, `claude`, `gemini`). 사용 가능한 모든 키는 [통합 목록](../integrations/index.md)을 참고하세요 |
| `--integration-options`  | 통합에 대한 옵션(예: `--integration-options="--commands-dir .myagent/cmds"`) |
| `--script sh\|ps\|py`    | 스크립트 유형: `sh`(bash/zsh), `ps`(PowerShell), `py`(Python)            |
| `--here`                 | 새 디렉터리를 만들지 않고 현재 디렉터리에서 초기화                       |
| `--force`                | 기존 디렉터리에서 초기화할 때 병합/덮어쓰기를 강제                       |
| `--allow-symlinks`       | 마이그레이션된 FileHelper 작업에서 프로젝트 내부 심볼릭 링크 허용을 영속화. 기본값은 거부 |
| `--ignore-agent-tools`   | AI 코딩 에이전트 CLI 도구 확인 건너뛰기                                  |
| `--preset <id>`          | 초기화 중에 프리셋 설치                                                  |

필요한 디렉터리 구조, 템플릿, 스크립트, AI 코딩 에이전트 통합 파일을 갖춘 새 Spec Kit 프로젝트를 만듭니다.

> [!NOTE]
> Git 저장소 초기화와 브랜치 작업은 기본으로 설치되지 않는 **git 확장**이 관리합니다. git 워크플로를 사용하려면 init 후에 `specify extension add git`을 실행하세요.

새 디렉터리를 만들려면 `<project_name>`을, 현재 디렉터리에서 초기화하려면 `--here`(또는 `.`)를 사용하세요. 디렉터리에 이미 파일이 있다면 `--force`로 확인 없이 병합할 수 있습니다.

`--integration`을 생략하면 대화형 터미널에서는 통합을 선택하라는 프롬프트가 표시됩니다. CI나 파이프 실행 같은 비대화형 세션은 GitHub Copilot을 기본값으로 사용합니다. 다른 통합을 명시적으로 선택하려면 `--integration <key>`를 전달하거나, 폴백을 바꾸려면 `SPECKIT_INTEGRATION_DEFAULT`를 설정하세요([환경 변수](#environment-variables) 참고).

### 영속화되는 파일시스템 정책

`init`만 `--allow-symlinks`를 받습니다. 이 옵션은 `.specify/init-options.json`에
`allow_symlinks: true`를 기록하며, 생략하면 재초기화 시를 포함해 `false`를
기록합니다. 프로젝트 범위 CLI 명령은 호출할 때마다 현재 값을 읽으므로 수동 편집이
즉시 반영됩니다. 파일이나 키가 없으면 거부를 뜻합니다. 값은 문자열, 숫자, null이
아니라 JSON 불리언이어야 합니다. 유효하지 않거나 읽을 수 없는 설정은 폴백이 아니라
오류입니다. 기록된 다른 init 옵션은 보존됩니다.

설정 부트스트랩 계층(`.specify`와 `init-options.json`)은 허용 모드에서도 신뢰되는
프로젝트 루트 아래에서 항상 심볼릭 링크가 아니어야 합니다. 도움말, 버전, check,
프로젝트 없는 도구/카탈로그 작업에는 프로젝트 요구 사항이 새로 생기지 않습니다.
init은 `SPECIFY_INIT_DIR`이 아니라 명시적인 대상을 사용합니다. `check`는 설치된
프로젝트 통합을 사용할 수 있습니다. 통합을 불러오거나 도구를 확인하기 전에, 선택된
cwd 또는 `SPECIFY_INIT_DIR` 경계에 존재하는 정책을 검증합니다. `.specify`나 init
옵션이 없는 프로젝트 없는 cwd에서도 여전히 동작합니다. 도움말과 버전은 프로젝트
정책을 읽지 않습니다.

**마이그레이션 한계:** 정책은 FileHelper 작업에서 적용되며, 현재는 init-options
부트스트랩 I/O가 포함됩니다. 정책을 불러온다고 해서 다른 모든 명령의 파일시스템
작업이 아직 FileHelper를 거치는 것은 아닙니다. 공유 인프라, 등록, 관리 스토리지,
스크립트, 런타임 작업은 마이그레이션되기 전까지 기존 동작을 유지합니다. 영속화된
설정이 모든 파일시스템 작업에서 이미 적용된다는 뜻은 아닙니다.

### 예시

```bash
# 통합을 지정해 새 프로젝트 만들기
specify init my-project --integration copilot

# 현재 디렉터리에서 초기화
specify init --here --integration copilot

# 비어 있지 않은 디렉터리에 강제 병합
specify init --here --force --integration copilot

# PowerShell 스크립트 사용(Windows/크로스 플랫폼)
specify init my-project --integration copilot --script ps

# 초기화 중에 프리셋 설치
specify init my-project --integration copilot --preset compliance
```

<a id="environment-variables"></a>

### 환경 변수

| 변수              | 설명                                                                     |
| ----------------- | ------------------------------------------------------------------------ |
| `SPECKIT_INTEGRATION_DEFAULT` | `--integration`을 생략했을 때 `specify init`이 사용하는 폴백 통합(대화형 프롬프트 기본값과 비대화형 폴백)을 재정의합니다. 등록된 통합 키(예: `gemini`, `claude`)로 설정하세요. 인식할 수 없는 값은 경고와 함께 무시되고 내장 기본값(`copilot`)이 사용됩니다. 명시적인 `--integration <key>`가 항상 우선합니다. |
| `SPECIFY_INIT_DIR` | 비대화형/CI 사용을 위해, `cd` 없이 디렉터리 밖(예: 모노레포 루트)에서 멤버 프로젝트를 대상으로 지정합니다. **프로젝트 루트**, 즉 `.specify/`를 *포함하는* 디렉터리로 설정하세요(상대 경로는 현재 디렉터리를 기준으로 해석됩니다). 경로가 존재하고 `.specify/`를 포함해야 하며, 그렇지 않으면 명령은 오류를 내고 현재 디렉터리로 폴백하지 **않습니다**. 코어 루트 헬퍼(Bash의 `get_repo_root`, PowerShell의 `Get-RepoRoot`)에서 한 번 해석되므로, 이를 상속하는 코어 기능 스크립트(`/speckit.plan`, `/speckit.tasks`, …)와 Git 확장의 기능 브랜치 생성에서도 적용됩니다. `specify` CLI는 모든 프로젝트 범위 하위 명령(`specify integration …`, `specify extension …`, `specify workflow …`, `specify preset …`, 그리고 `.specify/` 프로젝트에서 동작하는 나머지)에 **동일한** 검증 규칙을 적용하므로 이들도 멤버 프로젝트를 대상으로 삼을 수 있습니다. 설정하지 않으면 Bash/PowerShell 헬퍼는 기존의 상향 탐색을 유지하고, `specify` CLI는 명령이 더 넓은 감지를 명시적으로 정의하지 않는 한(예: bundle 명령) 프로젝트 범위 해석기를 cwd 전용으로 유지합니다. |
| `SPECIFY_FEATURE_DIRECTORY` | 활성 기능 디렉터리를 재정의합니다(`.specify/feature.json`보다 우선). 상대 경로는 포함 여부 검사 없이 프로젝트 루트에 결합되므로 `../shared-feature` 같은 값은 여전히 루트 밖으로 해석됩니다. 절대 경로는 그대로 사용되며 마찬가지로 프로젝트 루트 밖을 가리킬 수 있습니다. 프로젝트 루트는 여전히 Spec Kit 프로젝트와 그 명령/설정 컨텍스트를 식별하지만, 선택된 기능 디렉터리 자체가 그 아래에 있어야 하는 것은 아닙니다. `SPECIFY_INIT_DIR`과 함께 쓰면 프로젝트와 기능을 모두 비대화형으로 선택할 수 있습니다. |
| `SPECIFY_FEATURE` | 활성 기능 **레이블**(예: `001-photo-albums`)을 명시적으로 재정의합니다. 코어 헬퍼가 현재 기능/브랜치로 보고하는 식별자입니다(Bash의 `get_current_branch`, PowerShell의 `Get-CurrentBranch`). 이 헬퍼들은 Git을 전혀 살펴보지 않습니다. 변수가 설정되어 있으면 그대로 반환하고, 설정되어 있지 않으면 빈 문자열을 반환합니다. basename 폴백은 그 이후에 일어납니다. `get_feature_paths` / `Get-FeaturePathsEnv`가 해석된 기능 디렉터리의 basename으로 대체해 보고되는 레이블을 여전히 쓸 수 있게 하므로, 이름이 언급된 헬퍼를 직접 호출하면 그 폴백을 얻지 **못합니다**. 이 값은 직접 설정해야 합니다. 자식 프로세스는 부모의 환경을 바꿀 수 없으므로 Bash와 Python 기능 스크립트는 여러분이 실행할 주석 처리된 `export SPECIFY_FEATURE=…` / `$env:SPECIFY_FEATURE = …` 힌트를 **출력**하기만 하고, `/speckit.specify`는 이 변수를 설정하는 대신 `feature_directory`를 `.specify/feature.json`에 영속화합니다. (PowerShell 기능 스크립트는 `$env:SPECIFY_FEATURE`를 실제로 할당하지만, 스크립트가 현재 PowerShell 세션 안에서 실행될 때만 여러분에게 전달됩니다.) 이 변수는 기능 디렉터리를 찾아 주지 **않습니다**. `SPECIFY_FEATURE`만 설정하면 `get_feature_paths`가 *"Feature directory not found. Set `SPECIFY_FEATURE_DIRECTORY` or run the specify command to create `.specify/feature.json`."*으로 실패합니다. 디렉터리를 선택하려면 `SPECIFY_FEATURE_DIRECTORY`(위) 또는 `.specify/feature.json`을 사용하세요. |
| `SPECIFY_FEATURE_NO_PERSIST` | `1` 또는 `true`로 설정하면, 읽을 때 `SPECIFY_FEATURE_DIRECTORY`를 영속화했을 경우를 포함해 모든 코어 스크립트가 `.specify/feature.json`을 쓰지 않습니다. 여러 에이전트가 각자의 `SPECIFY_FEATURE_DIRECTORY`로 같은 체크아웃에서 동시에 실행될 때 유용합니다. 이 설정이 없으면 각 호출의 영속화 단계가 다른 에이전트가 고정한 기능 디렉터리를 덮어쓸 수 있습니다. |

> **두 가지 해석 축.** `SPECIFY_INIT_DIR`은 **프로젝트**(어느 디렉터리가 `.specify/`를 포함하는지)를 선택하고, `SPECIFY_FEATURE_DIRECTORY` / `.specify/feature.json`은 **기능**을 선택합니다. 기능의 기본 위치는 해당 프로젝트의 `specs/` 아래이지만, 명시적으로 선택한 기능 디렉터리가 그 안에 있어야 한다는 제한은 없습니다. 둘은 독립적이며, 프로젝트가 먼저이고 기능이 그다음입니다.
>
> **버전 관리.** `specify init`은 머신 로컬 상태, 즉 `feature.json`(기능을 전환할 때마다 다시 쓰이는 현재 기능 포인터)과 머신별 확장 `extensions/*/local-config.yml` 오버라이드를 제외하는 관리형 `.specify/.gitignore`를 스캐폴딩합니다. `.specify/` 아래의 나머지(컨스티튜션, 템플릿, 스크립트, 확장 설정)는 공유 가능하게 두어 팀이 정렬된 상태를 유지하도록 합니다. `.specify/`의 다른 공유 스크립트·템플릿과 마찬가지로 이 파일은 공유 인프라 매니페스트로 추적됩니다. 여러분의 편집은 재초기화 시 보존되고, `specify init --here --force`는 관리 내용을 복원합니다. `specify integration uninstall`은 제거되는 에이전트 자신의 파일만 삭제하므로 이 파일은 의도적으로 그대로 남습니다.
>
> **심볼릭 링크된 프로젝트 루트.** 명시적으로 설정된 프로젝트 루트 별칭은 그 위의
> OS 상위 경로를 포함해 신뢰됩니다. 그 루트 아래의 설정 계층은 항상 거부 모드로
> 검사됩니다. `.specify/`와 `init-options.json`은 심볼릭 링크일 수 없습니다.
> 프로젝트의 다른 곳에서 아직 마이그레이션되지 않은 작업은 기존 심볼릭 링크 동작을
> 유지합니다.

## 헬퍼 스크립트로 기능 이름 짓기

번들된 `create-new-feature` 헬퍼 스크립트를 직접 호출하면, 생성되는 이름은 UTF-8의
유니코드 문자와 십진 숫자를 유지하므로 `添加用户` 같은 설명은 `001-添加用户`를
만듭니다. 구두점만으로 이루어진 설명은 여전히 `001-` 같은 빈 접미사를 만들 수
있습니다. 이 경우 스크립트는 드라이 런 중에도 stderr로 경고합니다. JSON 출력은
계속 파싱 가능합니다.

다른 이름을 선택하려면 원래 설명을 유지하고 짧은 이름을 제공하세요.

```bash
bash .specify/scripts/bash/create-new-feature.sh --json --short-name 用户管理 "添加用户"
```

Python 헬퍼도 `--short-name`을 받고, PowerShell 헬퍼는 `-ShortName`을 사용합니다.
제공한 짧은 이름도 같은 규칙으로 정리되므로 문자나 숫자를 하나 이상 포함해야
합니다. 비ASCII 이름의 경우 Bash 헬퍼에는 설치된 UTF-8 로캘과 유니코드 분류를
위한 Python 3 인터프리터가 필요합니다. 탭과 줄바꿈을 포함한 ASCII 입력은 둘 다
없이도 정리됩니다. `LC_ALL`이 비어 있지 않으면 Bash는 다른 로캘을 선택하지 않고
그 로캘을 사용합니다. 선택된 로캘을 UTF-8 이름에 쓸 수 없으면 유니코드 이름은
오류로 실패합니다. `LC_ALL`이 설정되지 않았거나 비어 있으면, `LANG`이나
`LC_CTYPE`이 비UTF-8 로캘을 지정하더라도 Bash는 설치된 UTF-8 로캘을 선택합니다.
ASCII 대문자는 소문자로 바뀌고, 비ASCII 문자의 대소문자는 스크립트 변형 전반에서
보존됩니다.

## 설치된 도구 확인

```bash
specify check
```

CLI 기반 AI 코딩 에이전트를 시스템에서 사용할 수 있는지 확인합니다. IDE 기반 에이전트는 CLI 도구가 필요 없으므로 건너뜁니다.

설치된 프로젝트 통합이 있으면 포함됩니다. 선택된 프로젝트 경계에 존재하는
파일시스템 정책은 유효해야 합니다. 프로젝트 없는 cwd는 설정 없이도 계속
지원됩니다.

이 명령은 오프라인으로 유지됩니다. 명령이 이전 Spec Kit 버전처럼 동작하거나 기대한 CLI 기능이 없다면 `specify self check`를 실행해 로컬 CLI가 최신 릴리스보다 뒤처졌는지 확인하세요.

## 버전 정보

```bash
specify version
```

Spec Kit CLI 버전, Python 버전, 플랫폼, 아키텍처를 표시합니다.

네트워크 확인 없이 로컬 CLI 기능을 조회하려면 다음을 사용합니다.

```bash
specify version --features
```

전체 버전, 런타임, 시스템, 기능 정보를 JSON으로 출력하려면 다음을 사용합니다.

```bash
specify version --json
```

`--features`와 `--json`을 함께 쓰면 동일한 전체 JSON 출력이 나옵니다. JSON
모드에서 `--features`는 결과를 필터링하지 않습니다. OpenSSL 정보를 사용할 수
없으면 `runtime.openssl`은 `null`입니다.
성공한 JSON 출력은 stdout에만 기록됩니다. 실패 시에는 stdout을 비워 두고 정제된
JSON 오류 객체 하나를 stderr에 기록합니다.

다음으로도 빠르게 버전을 확인할 수 있습니다.

```bash
specify --version
specify -V
```

## 실험적 MCP 서버

```bash
specify mcp
```

실험적 stdio 전용 MCP 서버를 시작합니다. 일급 읽기 전용 도구는 버전
정보(`specify_version`), 아티팩트 목록과 합성(`specify_artifact_list`,
`specify_artifact_info`), 설치된 워크플로(`specify_workflow_list`)를 노출합니다.
범용 list, describe, run 도구는 안정적인 `version` JSON 명령만 지원합니다. 도구
이름, 결과 계약, 현재 제한 사항은 [MCP 서버 레퍼런스](mcp.md)를 참고하세요.
