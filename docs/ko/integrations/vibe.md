# Mistral Vibe

설정하기 전에 [Mistral Vibe](https://github.com/mistralai/mistral-vibe)를 설치하세요.
통합 키는 `vibe`입니다. 스킬 레이아웃에는 Mistral Vibe v2.0.0 이상이 필요합니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration vibe
```

기존 Spec Kit 프로젝트에서는 `specify integration install vibe`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.vibe/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.

생성된 스킬은 기본적으로 사용자와 모델 모두 호출할 수 있습니다. 선택적인
`--integration-options="--skills"` 플래그는 기본값을 다시 명시할 뿐입니다.

## 사용자 호출 전용 스킬

Mistral Vibe가 코어 Spec Kit 명령 스킬을 자동으로 호출하지 않게 하려면 다음과
같이 합니다.

```bash
specify init my-project --integration vibe --integration-options="--no-model-invocation"
```

이 옵션은 코어 명령 스킬에 `disable-model-invocation: true`를 기록합니다.
Mistral Vibe의 [README](https://github.com/mistralai/mistral-vibe#skills-system)에
따르면 그런 스킬은 명시적 호출 전용입니다. `/speckit-<command>`는 여전히 호출할 수
있지만, 모델은 그것을 자동으로 보거나 호출하지 않습니다.

- 확장 스킬은 모델 호출 가능 상태로 남습니다. 필수 확장 훅이 모델의 호출에
  의존하기 때문입니다. 확장의 필수 훅이 코어 명령을 실행한다면(`command:`가 코어
  `speckit.<command>`인 경우) 이 옵션을 끄세요. 코어 스킬이 사용자 호출 전용인
  동안에는 모델이 그 훅을 실행할 수 없습니다.
- 이 옵션은 통합의 다른 옵션과 함께 저장되고
  `specify integration upgrade vibe`에 의해 다시 적용됩니다. `upgrade`에
  `--integration-options`를 전달하면 저장된 모든 옵션이 대체됩니다.
  `--integration-options=""`는 옵션을 지웁니다. `specify init`을 다시 실행하는
  경우에도 마찬가지로 전달된 옵션만 저장됩니다.
- 프리셋이 오버라이드하거나 오버라이드를 중단한 코어 명령 스킬은, Mistral Vibe를
  재설치하지 않고 프리셋 스킬을 다시 등록하는 명령에 의해 기본값으로 되돌려집니다.
  프리셋을 설치·제거·업데이트하는 명령(프리셋을 담은 `specify bundle` 명령 포함),
  `specify integration use vibe`, 그리고 Mistral Vibe가 설치되어 있지만 기본값이
  아닐 때의 `specify integration switch vibe`가 해당합니다. 옵션을 다시 적용하려면
  Mistral Vibe가 기본 통합인 상태에서
  `specify integration upgrade vibe --force`를 실행하세요(그 명령들이 설치된 스킬
  파일을 수정하므로 `--force`가 필요하며, 이는 Spec Kit이 관리하는 파일에 대한
  로컬 편집을 덮어씁니다).

## 런타임 이벤트

네이티브 이벤트는 `.vibe/hooks.toml`을 사용합니다. Vibe는 `pre_tool_use`,
`post_tool_use`, `stop`만 지원하며, 이들은 `pre_tool`, `post_tool`, `post_agent`에
매핑됩니다. 세션 시작, 세션 종료, 프롬프트 제출 이벤트는 Vibe에 대해 내보내지지
않습니다. 지원되지 않는 핸들러는 경고와 함께 건너뜁니다.
`--integration-options="--events false"`는 Spec Kit 이벤트 핸들러를
비활성화합니다.
[런타임 이벤트](cli.md#runtime-events)를 참고하세요.

Spec Kit은 자신이 기록한, `speckit_marker = true`를 가진 `[[hooks]]` 항목만 새로
고치고 제거합니다. 각 항목은 마지막 TOML 줄과 그 바로 뒤의 빈 줄들에서 끝나므로,
여러분 자신의 훅 위에 적은 메모처럼 Spec Kit 항목 뒤에 오는 주석은 파일에
남습니다. Spec Kit의 항목을 제거한 뒤 주석과 빈 줄만 남으면 Spec Kit은
`.vibe/hooks.toml`을 삭제하며, 그 주석도 함께 삭제됩니다. Spec Kit은 Codex의
`.codex/config.toml`에도 같은 규칙을 적용합니다.

## 워크플로 디스패치

워크플로 스텝은 `vibe -p <prompt>`를 사용하고 구조화된 출력을 위해
`--output json`을 요청합니다. 어댑터는 워크플로의 `model` 설정을 무시합니다. 대신
Vibe의 `active_model` 설정이나 `VIBE_ACTIVE_MODEL`로 모델을 선택하세요.
