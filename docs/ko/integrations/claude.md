# Claude Code

설정하기 전에 [Claude Code](https://www.anthropic.com/claude-code)를 설치하세요.
통합 키는 `claude`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration claude
```

기존 Spec Kit 프로젝트에서는 `specify integration install claude`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.claude/skills/speckit-<command>/SKILL.md`로 설치됩니다.
Claude의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.

## 사용자 호출 전용 스킬

기본적으로 Claude Code용으로 설치된 모든 Spec Kit 스킬은 모델이 호출할 수 있으므로,
그 설명이 항상 Claude의 컨텍스트에 들어가고 스킬 목록 예산에 포함됩니다. 스킬이
많은 프로젝트에서는 이 때문에 다른 스킬이 목록에서 밀려날 수 있습니다. 코어 Spec
Kit 명령 스킬을 사용자 호출 전용으로 만들려면 다음과 같이 합니다.

```bash
specify init my-project --integration claude --integration-options="--no-model-invocation"
```

이 옵션은 코어 명령 스킬에 `disable-model-invocation: true`를 기록합니다.
Claude는 사용자가 `/speckit-<command>`를 호출할 때만 그 스킬을 실행할 수 있고, 그
설명은 컨텍스트에 들어가지 않습니다.

- 확장 스킬은 모델 호출 가능 상태로 남습니다. 필수 확장 훅이 모델의 호출에
  의존하기 때문입니다. 확장의 필수 훅이 코어 명령을 실행한다면(`command:`가 코어
  `speckit.<command>`인 경우) 이 옵션을 끄세요. 코어 스킬이 사용자 호출 전용인
  동안에는 모델이 그 훅을 실행할 수 없습니다.
- 이 옵션은 통합의 다른 옵션과 함께 저장되고
  `specify integration upgrade claude`에 의해 다시 적용됩니다. `upgrade`에
  `--integration-options`를 전달하면 저장된 모든 옵션이 대체되므로 유지하고 싶은
  옵션을 반복해 적으세요(예:
  `--integration-options="--events false --no-model-invocation"`).
  `--integration-options=""`는 옵션을 지웁니다. `specify init`을 다시 실행하는
  경우에도 마찬가지로 전달된 옵션만 저장됩니다.
- 프리셋이 오버라이드하거나 오버라이드를 중단한 코어 명령 스킬은, Claude를
  재설치하지 않고 프리셋 스킬을 다시 등록하는 명령에 의해 기본값으로 되돌려집니다.
  프리셋을 설치·제거·업데이트하는 명령(프리셋을 담은 `specify bundle` 명령 포함),
  `specify integration use claude`, 그리고 Claude가 설치되어 있지만 기본값이 아닐
  때의 `specify integration switch claude`가 해당합니다. 옵션을 다시 적용하려면
  Claude가 기본 통합인 상태에서
  `specify integration upgrade claude --force`를 실행하세요(그 명령들이 설치된
  스킬 파일을 수정하므로 `--force`가 필요하며, 이는 Spec Kit이 관리하는 파일에
  대한 로컬 편집을 덮어씁니다).

## 런타임 이벤트와 워크플로 디스패치

이 통합은 `.claude/settings.json`을 통해 네이티브 이벤트를 지원합니다.
Spec Kit 이벤트 핸들러를 비활성화하려면
`--integration-options="--events false"`를 사용하세요. 두 설정이 모두 필요하다면
`--no-model-invocation`과 함께 쓰세요.
[런타임 이벤트](cli.md#runtime-events)를 참고하세요.

워크플로 스텝은 `claude -p <prompt>`를 사용하며, 설정된 경우 `--model`을, 구조화된
출력이 요청된 경우 `--output-format json`을 함께 씁니다. 이 어댑터는 스스로 권한
우회 플래그를 추가하지 않습니다.

<a id="workflow-dispatch-settings"></a>

## 워크플로 디스패치 설정

명령 및 프롬프트 스텝은
[`reasoning_effort`](../workflow-steps/types.md#reasoning-effort-and-context-size)를
설정할 수 있으며, Claude Code 디스패치는 이를 `--effort <value>`로 전달합니다.
Claude Code 2.1.318은 `low`, `medium`, `high`, `xhigh`, `max`를 받습니다. 그 밖의
값은 경고만 내고 무시하므로, Spec Kit은 Claude Code가 시작되기 전에 다른 값을
거부합니다. 더 새로운 Claude Code 릴리스가 추가한 수준을 전달하려면
`reasoning_effort` 대신 `SPECKIT_INTEGRATION_CLAUDE_EXTRA_ARGS="--effort <value>"`를
설정하세요. 이 환경 변수는 한 스텝만이 아니라 모든 Claude Code 호출에 적용됩니다.

Spec Kit은 Claude Code에 대해 `context_size`를 매핑하지 않으므로 이를 설정한
스텝은 실패합니다. 대신 스텝의 `model` 필드로 긴 컨텍스트 모델을 선택하세요.
