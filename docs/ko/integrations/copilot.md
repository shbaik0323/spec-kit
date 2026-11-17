# GitHub Copilot

에디터에서 [GitHub Copilot](https://code.visualstudio.com/)을 설정하거나 Copilot
CLI를 사용하세요. 통합 키는 `copilot`이며, IDE 설정에는 CLI가 필요하지 않습니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있지 않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

<a id="setup"></a>

## 설정

```bash
specify init my-project --integration copilot
```

기존 Spec Kit 프로젝트에서는 `specify integration install copilot`을 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

<a id="default-skills-layout"></a>

## 기본 스킬 레이아웃

스킬은 `.github/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`로 호출하세요. 이들은 터미널 명령이 아니라 에이전트
스킬입니다. 생성된 스킬은 기본적으로 사용자와 모델 모두 호출할 수 있습니다.

## 사용자 호출 전용 스킬

스킬 레이아웃에서 Copilot이 코어 Spec Kit 명령 스킬을 자동으로 로드하지 않게
하려면 다음과 같이 합니다.

```bash
specify init my-project --integration copilot --integration-options="--no-model-invocation"
```

이 옵션은 코어 명령 스킬에 `disable-model-invocation: true`를 기록합니다.
VS Code의 [에이전트 스킬 문서](https://code.visualstudio.com/docs/agent-customization/agent-skills)는
이렇게 하면 `/` 슬래시 명령을 통한 수동 호출만 가능하다고 설명합니다.
GitHub의 Copilot CLI 스킬 문서에는 이 필드가 나와 있지 않지만, Copilot CLI
1.0.70으로 수동 확인한 결과 플래그가 없을 때는 모델이 `speckit-specify`에 대해
`skill` 도구를 호출했고 플래그가 있을 때는 호출하지 않았으며, 명시적인
`/speckit-specify`는 여전히 실행되었습니다. 플래그가 있어도 모델은 스킬의
`SKILL.md`를 일반 파일로 읽을 수 있었으므로 완전한 차단은 아닙니다. 어느 쪽 문서도
이 플래그를 가진 스킬의 설명이 모델의 컨텍스트에서 빠지는지는 밝히지 않습니다.

- 이 옵션은 `--commands`로 명시적으로 선택했든, 레이아웃 플래그 없이 기존
  설치에서 유지되었든 명령 레이아웃에서는 효과가 없습니다. `.agent.md`와
  `.prompt.md` 파일은 스킬이 아니므로 플래그가 기록되지 않습니다.
- 확장 스킬은 모델 호출 가능 상태로 남습니다. 필수 확장 훅이 모델의 호출에
  의존하기 때문입니다. 확장의 필수 훅이 코어 명령을 실행한다면(`command:`가 코어
  `speckit.<command>`인 경우) 이 옵션을 끄세요. 코어 스킬이 사용자 호출 전용인
  동안에는 모델이 그 훅을 실행할 수 없습니다.
- 이 옵션은 통합의 다른 옵션과 함께 저장되고
  `specify integration upgrade copilot`에 의해 다시 적용됩니다. `upgrade`에
  `--integration-options`를 전달하면 저장된 모든 옵션이 대체됩니다.
  `--integration-options=""`는 옵션을 지웁니다. `specify init`을 다시 실행하는
  경우에도 마찬가지로 전달된 옵션만 저장됩니다.
- 프리셋이 오버라이드하거나 오버라이드를 중단한 코어 명령 스킬은, Copilot을
  재설치하지 않고 프리셋 스킬을 다시 등록하는 명령에 의해 기본값으로 되돌려집니다.
  프리셋을 설치·제거·업데이트하는 명령(프리셋을 담은 `specify bundle` 명령 포함),
  `specify integration use copilot`, 그리고 Copilot이 설치되어 있지만 기본값이 아닐
  때의 `specify integration switch copilot`이 해당합니다. 옵션을 다시 적용하려면
  Copilot이 기본 통합인 상태에서
  `specify integration upgrade copilot --force`를 실행하세요(그 명령들이 설치된
  스킬 파일을 수정하므로 `--force`가 필요하며, 이는 Spec Kit이 관리하는 파일에
  대한 로컬 편집을 덮어씁니다).

<a id="commands-layout"></a>

## 명령 레이아웃

지원되는 명령 레이아웃을 선택하려면 다음과 같이 합니다.

```bash
specify init my-project --integration copilot --integration-options="--commands"
```

이렇게 하면 `.github/agents/` 아래에 `.agent.md` 파일을, `.github/prompts/` 아래에
짝이 되는 `.prompt.md` 파일을 설치하고 `.vscode/settings.json`을 병합합니다.
이 레이아웃에서 Copilot CLI는 `/agents`를 통해 에이전트를 선택하거나 프롬프트에서
직접 지목할 수 있습니다.

`specify integration upgrade copilot`으로 기존 명령 설치를 마이그레이션할 때 기본
스킬 레이아웃을 명시적으로 선택하려면 `--integration-options="--skills"`를
사용하세요. 이를 전달하면 저장된 모든 옵션이 대체되므로 유지하고 싶은 옵션을
포함하세요. 예를 들어 그 값에 `--no-model-invocation`이 없으면 마이그레이션된
스킬은 모델 호출 가능으로 기록됩니다. 프리셋 아티팩트가 등록된 상태에서의
레이아웃 변경 제한을 포함한
[공통 업그레이드 규칙](cli.md#upgrade-an-integration)을 참고하세요.

`--skills`와 `--commands`는 함께 쓸 수 없습니다. 명시적인 레이아웃 플래그가
없으면 기존의 관리되는 명령 설치는 업그레이드나 활성화 시 레이아웃을 유지하고,
새 설치는 스킬을 기본값으로 합니다.

명령 레이아웃은 누락된 VS Code 설정과 하위 키만 추가합니다. 기존 값은 변경되지
않습니다. 기존 설정 파일이 JSONC이거나 그 밖의 이유로 JSON 객체로 파싱할 수 없으면
경고 및 수동 지침과 함께 병합을 건너뜁니다. 병합된 기존 설정 파일은 제거 시
유지되고, 새로 만들어진 설정 파일은 매니페스트로 추적됩니다.

## 런타임 이벤트와 워크플로 권한

네이티브 이벤트는 `.github/hooks/speckit.json`을 사용합니다.
`--integration-options="--events false"`는 Spec Kit 이벤트 핸들러를
비활성화합니다.
[런타임 이벤트](cli.md#runtime-events)를 참고하세요.

워크플로 스텝에는 IDE 설정과 별도로 Copilot CLI가 필요합니다. 스킬 모드에서 명령
스텝은 프롬프트에 `/speckit-<command>`를 전달합니다. 명령 모드에서는 에이전트
파일을 슬래시 명령으로 취급하는 대신 `--agent speckit.<command>`를 선택하고
사용자의 인자를 프롬프트로 전달합니다.

헤드리스 디스패치는 기본적으로 `--yolo`를 추가해 대화형 승인 없이 도구, 경로,
URL을 허용합니다. Spec Kit이 이 플래그를 추가하지 않게 하려면
`SPECKIT_COPILOT_ALLOW_ALL_TOOLS=0`을 설정하세요. 더 이상 권장되지 않는
`SPECKIT_ALLOW_ALL_TOOLS`는 폴백으로만 받아들여집니다. 설정된 `--model`과 요청된
`--output-format json`은 전달됩니다.

<a id="workflow-dispatch-settings"></a>

## 워크플로 디스패치 설정

명령 및 프롬프트 스텝은
[`reasoning_effort`와 `context_size`](../workflow-steps/types.md#reasoning-effort-and-context-size)를
설정할 수 있습니다.
Copilot CLI 디스패치는 스킬 레이아웃과 명령 레이아웃 모두에서 이를 `--model` 뒤에
네이티브 플래그로 전달합니다.

| 스텝 설정 | Copilot CLI 플래그 | Copilot CLI 1.0.106의 값 |
|---|---|---|
| `reasoning_effort` | `--reasoning-effort <value>` | `none`, `minimal`, `low`, `medium`, `high`, `xhigh`, `max` |
| `context_size` | `--context <value>` | `default`, `long_context` |

Spec Kit은 이 값들만 받습니다. 그 밖의 값은 Copilot CLI가 시작되기 전에 스텝을
실패시킵니다. Copilot CLI가 거부한 값을 오류에 그대로 반복하고, 그 값이 캡처된
출력에서 올 수 있기 때문입니다. 더 새로운 Copilot CLI 릴리스가 추가한 값을
전달하려면 스텝 설정 대신
`SPECKIT_INTEGRATION_COPILOT_EXTRA_ARGS="--reasoning-effort <value>"` 또는
`SPECKIT_INTEGRATION_COPILOT_EXTRA_ARGS="--context <value>"`를 설정하세요.
