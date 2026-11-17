# 통합 API 레퍼런스

이 레퍼런스는 내장(임베디드) 통합과 외부 통합 어댑터를 모두 작성하기 위한
지원되는 Python 계약을 설명합니다. 둘은 같은 기반 클래스, 렌더링 훅, 런타임
디스패치, 파일 소유권 API를 사용합니다. 디스크립터와 카탈로그 스키마는
[카탈로그 레퍼런스](catalog.md)에, 설치·신뢰·복구 명령은
[CLI 레퍼런스](cli.md)에 문서화되어 있습니다.

내부 레지스트리, 로더, 트랜잭션 헬퍼를 임포트하지 말고 아래의 기반 클래스와 훅을
사용하세요. 그런 구현 세부 사항은 지원되는 작성 API의 일부가 아닙니다.

## 전달 경로

내장 통합은 Specify CLI와 함께 제공되며 독립형 어댑터 디스크립터나 카탈로그
다운로드가 필요하지 않습니다. 소스 등록과 전달 요구 사항은 저장소의
[기여자 안내](https://github.com/github/spec-kit/blob/main/CONTRIBUTING.md#integration-pull-requests)를
따릅니다.

외부 어댑터는 독립적으로 게시되는 패키지입니다. 같은 작성 API를 구현하지만 아래의
패키지 검증, 프로젝트 로컬 경로, 실행 동의 요구 사항도 충족해야 합니다. 전달
경로가 비공개 구현 헬퍼를 공개로 만들지는 않습니다.

<a id="package-contract"></a>

## 패키지 계약

이 섹션은 외부 어댑터에만 적용됩니다.

루트에 다음 파일을 담은 ZIP, tar.gz, tgz 아카이브를 게시하세요.

```text
sample-agent/
├── integration.yml
├── __init__.py
└── helpers.py       # optional package-relative modules
```

아카이브를 감싸는 디렉터리 하나는 허용됩니다. 루트 모듈은 `key`가 디스크립터의
`integration.id`와 일치하는 구체적인 `IntegrationBase` 하위 클래스를 정확히 하나
내보내야 합니다. 어댑터 자신의 패키지로부터의 상대 임포트가 지원됩니다. 임포트
부수 효과로 클래스를 등록하지 마세요.

호스트 API와 Python 표준 라이브러리를 사용하는 어댑터는 별도의 pip 설치도, Spec
Kit 소스 레지스트리의 변경도 필요하지 않습니다. 호스트의 명령과 확장/프리셋 기여를
적응시키세요. 코어 명령 템플릿을 어댑터 소유의 목록으로 재배포하지 마세요.

## 형식 기반 클래스

다음 클래스를 `specify_cli.integrations.base`에서 임포트하세요.

| 기반 클래스 | 설치 형식 | 일반적인 명령 경로 |
| --- | --- | --- |
| `MarkdownIntegration` | Markdown 명령 | `commands/speckit.plan.md` |
| `TomlIntegration` | TOML 명령 | `commands/speckit.plan.toml` |
| `YamlIntegration` | YAML 레시피 | `recipes/speckit.plan.yaml` |
| `SkillsIntegration` | 에이전트 스킬 | `skills/speckit-plan/SKILL.md` |
| `IntegrationBase` | 사용자 정의 레이아웃 | 어댑터가 정의 |

표의 경로는 설정된 에이전트 폴더 아래의 예시입니다. 형식 기반 클래스는 호스트의
템플릿과 선택된 스크립트 변형을 렌더링합니다. 순수한 `IntegrationBase.setup()`은
원시 템플릿을 복사합니다. 렌더링을 직접 구현하지 않는 한 형식 기반 클래스를
우선하세요.

### 최소 스킬 어댑터

```python
from specify_cli.integrations.base import SkillsIntegration


class SampleIntegration(SkillsIntegration):
    key = "sample-agent"
    config = {
        "name": "Sample Agent",
        "folder": ".sample-agent",
        "commands_subdir": "skills",
        "install_url": "https://example.com/sample-agent",
        "requires_cli": False,
    }
    registrar_config = {
        "dir": ".sample-agent/skills",
        "format": "markdown",
        "args": "$ARGUMENTS",
        "extension": "/SKILL.md",
    }
    multi_install_safe = True
```

외부 패키지라면 이 모듈을 ID와 이름이 `sample-agent`와 `Sample Agent`이고
`requires.speckit_version`이 여러분이 테스트한 호스트 버전과 일치하는
`integration.yml`과 짝지으세요.
내장 통합은 패키지 디스크립터 없이 같은 클래스 메타데이터를 사용합니다. 이 예시는
IDE 기반 에이전트용 스킬을 설치합니다. 비대화형 CLI 디스패치는 지원하지 않습니다.

## 클래스 메타데이터

| 속성 | 계약 |
| --- | --- |
| `key` | 고유한 통합 식별자. 외부 어댑터는 `integration.id`와 일치해야 하며 내장 키를 대체할 수 없습니다 |
| `config.name` | 표시 이름. 외부 어댑터는 `integration.name`과 일치해야 합니다 |
| `config.folder` | 정식 프로젝트 상대 에이전트 루트 |
| `config.commands_subdir` | 그 루트 아래의 명령 또는 스킬 디렉터리 |
| `config.install_url` | 에이전트 설치 문서 URL |
| `config.requires_cli` | 에이전트에 실행 파일이 필요한지 나타내는 불리언 |
| `registrar_config.dir` | `config.folder/config.commands_subdir`와 일치 |
| `registrar_config.format` | 렌더링 형식: `markdown`, `toml`, `yaml` |
| `registrar_config.args` | `$ARGUMENTS`나 `{{args}}` 같은 에이전트 인자 자리 표시자 |
| `registrar_config.extension` | 점으로 시작하는 일반 파일명 접미사, 또는 Markdown 스킬의 경우 `/SKILL.md` |
| `invoke_separator` | 비어 있지 않은 호출 구분자. 기본값은 `.`이며 `SkillsIntegration`은 `-` |
| `dev_no_symlink` | 불리언. 개발 등록에서 심볼릭 링크 대신 파일 출력을 요청 |
| `multi_install_safe` | 불리언, 기본값 false. true는 정적이고 겹치지 않는 출력 루트를 요구하거나 파일이 없어야 합니다 |
| `version` | 선택적 클래스 속성. 외부 어댑터에서 제공하는 경우 디스크립터 버전과 일치해야 합니다 |

선택적인 `registrar_config.invoke_separator`는 등록 시 클래스 구분자를
재정의합니다. 개발용 심볼릭 링크는 클래스나 `registrar_config.dev_no_symlink` 중
어느 쪽이든 요청하면 비활성화됩니다. 선택적인 `registrar_config.legacy_dir`은
마이그레이션을 위한 이전의 프로젝트 상대 출력 디렉터리를 선언합니다.

외부 어댑터의 경우 출력 루트는 홈 상대 경로일 수 없고, `.git`이나 `.specify`로
들어갈 수 없으며, 경로 순회를 포함하거나 심볼릭 링크를 통과할 수 없습니다. 기본
대상과 레거시 대상은 서로 겹칠 수 없습니다. 다중 설치 안전 어댑터는 다른 통합의
대상과도 겹칠 수 없습니다. 비교는 모든 플랫폼에서 대소문자를 구분하지 않습니다.
일부 내장 통합에는 호스트가 관리하는 홈 범위 또는 레거시 레이아웃이 있습니다.
그것이 외부 패키지 계약을 완화하지는 않습니다.

## 설치 시 옵션과 레이아웃

클래스 메서드 `options() -> list[IntegrationOption]`을 오버라이드해 CLI의
`--integration-options`로 받는 플래그를 선언하세요. `IntegrationOption`은
`specify_cli.integrations.base`에서 임포트합니다.

| `IntegrationOption` 필드 | 의미 |
| --- | --- |
| `name` | `--skills` 같은 플래그 표기 |
| `is_flag` | true이면 불리언 스위치. 그렇지 않으면 값을 받음 |
| `required` | 호출자가 옵션을 반드시 제공해야 하는지 여부 |
| `default` | 생략했을 때의 값. `None`은 기본값 없음을 뜻함 |
| `help` | 사용자에게 보이는 옵션 설명 |

이들은 설치 옵션이며 스텝별 워크플로 런타임 옵션이 아닙니다.
자신의 옵션을 추가할 때는 상속받은 옵션을 보존하세요.

둘 이상의 레이아웃을 지원하는 어댑터는 다음 훅을 일관되게 유지하세요.

| 훅 | 계약 |
| --- | --- |
| `is_skills_mode(parsed_options=None, project_root=None) -> bool` | 스킬과 명령 중 선택. `SkillsIntegration`은 기본적으로 true를 반환 |
| `effective_invoke_separator(parsed_options=None, project_root=None) -> str` | 선택된 설치 옵션에 대한 호출 구분자 |
| `invoke_separator_for_mode(skills_enabled: bool) -> str` | 해석된 설치 모드에 대한 명령 참조 구분자 |
| `command_filename(template_name: str) -> str` | 소스 템플릿 스템에 대한 대상 파일명 |
| `build_command_invocation(command_name: str, args: str = "") -> str` | 코어 또는 기여된 명령에 대한 네이티브 에이전트 호출 |

기존 프로젝트의 라이프사이클 작업 중에 옵션이 없을 때, `project_root`는 모드를
인식하는 어댑터가 설치된 레이아웃을 조용히 마이그레이션하는 대신 살펴볼 수 있게
해 줍니다. 호출 표기, 파일 이름 지정, 렌더링은 서로 일치해야 합니다.

## 명령 렌더링 훅

`process_registered_command(frontmatter, body, *, source_id)`는 명령이 렌더링되기
전에 `(frontmatter, body)` 쌍을 반환합니다. `source_id`는 `core`이거나 기여하는
확장/프리셋 ID입니다. 스크립트 경로는 이미 조정된 상태이고, 인자 자리 표시자와
명령 참조 토큰은 그 뒤에 해석됩니다. 프런트매터 `name`은
`registrar_config.inject_name`이 활성화되어 있고 비어 있지 않은 이름이 남아 있지
않을 때만 주입됩니다. `frontmatter["name"]`이 존재한다고 가정하지 마세요. 입력
프런트매터를 변경하지 말고 새 객체를 반환하세요.

`post_process_command_content(content: str) -> str`은 렌더링된 Markdown, TOML,
YAML 명령 텍스트를 변환합니다. 기본 구현은 내용을 그대로 반환합니다.

`SkillsIntegration.post_process_skill_content(content: str) -> str`은 생성된 스킬
내용을 사용자 정의합니다. 상속받은 `SkillsIntegration.setup()`과 확장 및 프리셋
스킬 생성 패스가 스킬을 생성하거나 다시 쓸 때 이를 호출합니다. 기반 구현은 점
표기 훅 명령 이름을 에이전트 네이티브의 하이픈 호출로 변환하기 위한 공통 안내를
추가합니다. 오버라이드할 때는 예를 들어 `super()`를 호출해 그 안내를 보존하세요.

`process_registered_command()`와 `post_process_command_content()`는 확장 및 프리셋
기여를 포함한 명령 등록 중에 적용됩니다. 상속받은
`MarkdownIntegration.setup()`, `TomlIntegration.setup()`,
`YamlIntegration.setup()`은 어느 훅도 호출하지 않고 코어 템플릿을 설치합니다. 코어
명령에 추가적인 에이전트별 변환이 필요하다면, 등록 훅에만 의존하지 말고
`setup()`을 오버라이드해 [추적되는 쓰기](#setup-and-file-ownership)를 사용하세요.

이후의 스킬 생성 패스는 앞선 명령 등록 패스가 만든 것을 포함해 이미 존재하는
`SKILL.md`를 건너뛸 수 있습니다. 건너뛰는 패스에서는 그 파일에 대해 훅 호출이
일어나지 않습니다. 이후 패스가 설치된 모든 스킬을 다시 처리한다고 가정하지
마세요. 이 훅들은 Spec Kit이 생성한 출력에 관한 것이며, 에이전트나 사용자가 제공한
관련 없는 스킬에 관한 것이 아닙니다.

호스트 렌더링 코드를 복사하기보다 이 훅들을 우선하세요. 생성되는 PowerShell
기능은 Windows PowerShell 5.1과 호환되어야 합니다.

## 비대화형 CLI 디스패치

CLI 어댑터는 이 훅을 구현하며, 셸 명령 문자열이 아니라 인자 벡터를 반환합니다.

```python
def build_exec_args(
    self,
    prompt: str,
    *,
    model=None,
    output_json=True,
    integration_args=None,
    integration_options=None,
    project_root=None,
) -> list[str] | None:
    ...
```

반환되는 목록은 실행 파일과 그 뒤의 인자를 포함합니다. 디스패치가 지원되지 않으면
`None`을 반환하세요. 명시적으로 선언한 호스트 키워드 매개변수에는 기본값이 있어야
합니다. 생략된 키워드를 `**kwargs`로 받는 것도 지원됩니다. 프롬프트만 있는 호출과
완전한 호스트 호출이 모두 유효해야 합니다.

상속받는 CLI 디스패치는 기반 클래스마다 다릅니다.

| 기반 클래스 | 기본 디스패치 |
| --- | --- |
| `MarkdownIntegration`, `SkillsIntegration` | `config.requires_cli`가 true이면 `-p <prompt>`, 제공된 경우 `--model <model>`, `output_json`이 true이면 `--output-format json`을 사용 |
| `TomlIntegration` | 같은 프롬프트 및 JSON 플래그를 쓰지만 모델 플래그는 `-m <model>`. 마찬가지로 `config.requires_cli` 필요 |
| `IntegrationBase`, `YamlIntegration` | `None` 반환. 기본 CLI 디스패치 없음 |

에이전트가 다른 플래그나 다른 실행 파일을 사용한다면 `build_exec_args()`를
오버라이드하세요. 명령 파일을 설치하는 것만으로는 비대화형 디스패치 지원이
제공되지 않습니다.

`validate_runtime_config(integration_args=None, integration_options=None)`은
스텝별 인자 목록과 옵션 매핑을 검증합니다. 기본 구현은 비어 있지 않은 런타임
설정을 `ValueError`로 거부합니다. 그 값을 지원하는 어댑터는 지원되지 않는 옵션을
조용히 무시하지 말고 `build_exec_args()`에서 검증하고 변환해야 합니다.

외부 어댑터의 경우 디스크립터의 필수 도구는 `PATH`에서 확인됩니다. CLI 어댑터가
아무것도 선언하지 않으면 도구 확인은 그 런타임 실행 파일을 탐침합니다.
버전 감지는 에이전트별입니다. 버전 제약을 선언한다고 해서 보편적인 `--version`
호출이 제공되지는 않습니다.

<a id="per-step-dispatch-settings"></a>

### 스텝별 디스패치 설정

워크플로 `command` 및 `prompt` 스텝은 CLI 호출 한 번에 대해
[`reasoning_effort`와 `context_size`](../workflow-steps/types.md#reasoning-effort-and-context-size)를
요청할 수 있습니다. `specify_cli.integrations.base`의 `DISPATCH_SETTINGS`가
확인되는 순서대로 이들을 나열합니다. 값은 의미가 에이전트의 CLI에 속하는 불투명한
문자열입니다.

위의 `build_exec_args()` 시그니처는 바뀌지 않으며, 참여를 선언하지 않은 어댑터는
어느 키워드도 받지 않습니다. 설정을 요청하는 스텝은 CLI가 실행되기 전에
실패합니다. 프롬프트 스텝은 요청된 설정을 `build_exec_args()`에 전달합니다. 명령
스텝은 `dispatch_command()`에 전달하며, 그 기반 구현이 이를 `build_exec_args()`로
전달합니다. 따라서 *수신 메서드*는 `build_exec_args()`이거나, 통합이 오버라이드한
경우 명령 스텝에 대해서는 `dispatch_command()`입니다. 참여를 선언하려면 다음과
같이 합니다.

- 수신 메서드를 정의하는 클래스에 `supported_dispatch_settings`, 즉 통합이
  변환하는 설정들의 `frozenset`을 선언합니다. 그 메서드를 오버라이드하는 하위
  클래스는 지원을 다시 선언해야 합니다. 다른 클래스에 한 선언이나 set 또는
  frozenset이 아닌 선언은 아무것도 선언하지 않습니다.
- 선언한 설정마다 수신 메서드에서 `reasoning_effort=None` 또는
  `context_size=None` 키워드를 받습니다. 호스트는 스텝이 그 설정을 지정할 때만
  키워드를 전달하며 `""`는 절대 전달하지 않습니다. 값을
  `SPECKIT_INTEGRATION_<KEY>_EXTRA_ARGS`에서 온 인자 뒤에 놓이는 CLI 인자로
  변환하고, 그대로 넘기지 마세요. `IntegrationBase.build_exec_args()`와 형식 기반
  빌더는 어느 설정도 변환하지 않으며 값을 버리는 대신 `DispatchSettingsError`를
  발생시킵니다.
- 선택적으로 `validate_dispatch_setting(name, value)`를 오버라이드해 CLI가
  무시하거나 잘못 읽을 값에 대해 `ValueError`를 발생시키고,
  `dispatch_setting_hint(setting)`을 오버라이드해 호스트가 지원되지 않는 설정에
  대한 오류에 덧붙일 해결 안내 텍스트를 반환할 수 있습니다.

CLI가 실행되기 전에 호스트 게이트 `check_dispatch_settings()`가 요청된 각 설정을
`DISPATCH_SETTINGS` 순서로 확인합니다. 값은 문자나 숫자로 시작하는 1~64자의 문자,
숫자, `.`, `_`, `-`여야 하고, 통합이 위와 같이 그 설정을 선언해야 하며, 수신
메서드가 그 키워드나 `**kwargs`를 받아야 합니다. 그런 다음 게이트는
`validate_dispatch_setting()`을 호출합니다. 각 실패는 `setting` 속성이 설정의
이름을 담은 `ValueError`인 `DispatchSettingsError`를 발생시킵니다. 수신 메서드
자체가 값을 거부해야 한다면 `setting`과 함께 이를 발생시키세요. 값은 어떤 오류
메시지에도 넣지 마세요. 값은 캡처된 출력에서 올 수 있으며, Spec Kit은 그것을
터미널에 출력하거나 `log.jsonl`에 기록하지 않습니다. 호스트는 수신 메서드가
받아 놓고(예: `**kwargs`를 통해) 무시하는 선언된 설정을 감지할 수 없으므로, 선언은
그것을 변환하겠다는 약속입니다.

<a id="setup-and-file-ownership"></a>

## 설정과 파일 소유권

형식 기반 클래스가 설정과 해체를 구현합니다. 사용자 정의 레이아웃은 다음을
오버라이드할 수 있습니다.

| 훅 | 계약 |
| --- | --- |
| `preflight_setup(project_root, parsed_options=None, *, script_type=None) -> None` | 공유 파일이 기록되기 전에 설정을 거부. `ValueError` 또는 `OSError` 발생 |
| `setup(project_root, manifest, parsed_options=None, **opts) -> list[Path]` | 에이전트 파일을 쓰고 기록. 생성된 경로 반환 |
| `teardown(project_root, manifest, *, force=False) -> tuple[list[Path], list[Path]]` | 제거된 경로와 보존된 경로 반환. 상속된 동작은 강제하지 않는 한 수정된 파일을 보존 |
| `stale_cleanup_exclusions() -> set[str]` | 업그레이드가 오래된 것으로 삭제해서는 안 되는 프로젝트 상대 경로 |

적절한 경우 상속된 동작을 호출하고 `script_type`, `events`, `force`를 포함해
호스트가 제공하는 옵션을 보존하세요. 생성된 파일은 선언된 에이전트 루트 아래에
두세요. 공유 Spec Kit 스크립트와 템플릿은 호스트가 소유합니다.

호스트는 설정과 해체에 `IntegrationManifest`를 제공합니다. 사용자 정의 쓰기에는
다음의 지원되는 작업을 사용하세요.

| 작업 | 계약 |
| --- | --- |
| `manifest.record_file(rel_path, content, *, ownership="whole") -> Path` | 바이트 또는 UTF-8 텍스트를 쓰고 소유권을 기록. 경로는 프로젝트 상대 |
| `manifest.record_existing(rel_path, *, recovered=False, ownership=None) -> None` | 기존 일반 파일을 기록. `recovered=True`는 보존된 기존 내용임을 표시 |
| `IntegrationBase.write_file_and_record(content, dest, project_root, manifest) -> Path` | LF 줄바꿈으로 UTF-8 텍스트를 쓰고 기록. `dest`는 대상 경로 |

소유권은 어댑터가 소유한 파일이면 `whole`, 파일에 병합된 항목이면 `partial`,
호스트 공유 아티팩트이면 `shared`입니다. 사용자 설정이나 공유 디스패처에 대해 파일
전체 소유권을 주장하지 마세요. 매니페스트 경로는 정식이고, 프로젝트 로컬이며,
심볼릭 링크가 아니어야 합니다.

**기존 내용을 덮어쓰기 전에** 쓰고 기록하는 작업을 사용하세요.
`record_existing()`만으로는 호스트의 추적되는 쓰기 경로 밖에서 이미 덮어쓴 바이트를
복원할 수 없습니다. 그런 실패는 보고되고 결과 파일은 수동 복구를 위해 유지됩니다.

<a id="external-adapter-lifecycle-guarantees"></a>

## 외부 어댑터 라이프사이클 보장

이 섹션의 패키지 신뢰와 재승인은 외부 어댑터에 적용되며, CLI와 함께 제공되는 내장
통합에는 적용되지 않습니다.

설치하려면 실행 가능한 Python을 다운로드/임포트하기 전에 설치가 활성화된
카탈로그와 로컬 신뢰 결정이 필요합니다. 메타데이터 전용 조회 명령은
`specify integration list --catalog`, `specify integration search`,
`specify integration info`입니다. 일반 `specify integration list`는 신뢰된 설치
어댑터 코드를 로드합니다. 임포트 없는 조회 명령이 아닙니다.

동의는 프로젝트, 통합 ID, 검증된 패키지 내용에 묶입니다. 프로젝트를 복사하거나,
사용자를 바꾸거나, 패키지 바이트를 바꾸면 새로운 승인이 필요합니다. 없거나,
수정되었거나, 호환되지 않거나, 임포트할 수 없는 패키지는 명시적으로 실패합니다.
설치된 코드를 편집하지 말고 새 아카이브를 게시해 업그레이드하세요.

실패한 라이프사이클 작업은 관련 없는 파일, 워크플로 진행 상황, 충돌하는 외부
편집을 보존하면서 자신이 추적한 변경을 복원합니다. 복구 실패는 수동 복구를 위해
유지된 스냅샷을 보고합니다. 업그레이드와 제거는 강제하지 않는 한 수정된 생성 파일을
보존합니다. 강제 복구도 오래된 파일을 삭제하려면 여전히 로컬 소유권 증명이
필요하며 카탈로그 정책이나 대체 패키지 신뢰를 우회하지 않습니다.

어댑터 인스턴스는 동시 디스패치와 렌더링 작업을 처리할 수 있습니다. 호출별 상태는
로컬로 유지하고, 변경 가능한 전역 등록이나 다른 프로젝트에 설치된 패키지에
의존하지 마세요. 네이티브 이벤트 지원의 경우 `emit_events()`나
`remove_events()`를 확장할 때 호스트의 이벤트 소유권을 보존하세요. 지원되는
이벤트와 옵션은 [런타임 이벤트](cli.md#runtime-events)를 참고하세요.

## 함께 보기

- [통합 개발 및 게시](contributing.md)
- [디스크립터 및 카탈로그 스키마](catalog.md)
- [통합 CLI와 신뢰](cli.md)
- [스텝별 워크플로 통합 설정](../workflow-steps/types.md#per-step-integration-configuration)
