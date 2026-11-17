# Codex CLI

설정하기 전에 [Codex CLI](https://github.com/openai/codex)를 설치하세요.
통합 키는 `codex`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration codex
```

기존 Spec Kit 프로젝트에서는 `specify integration install codex`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.agents/skills/speckit-<command>/SKILL.md`로 설치됩니다.
Codex의 채팅에서 달러 접두사로 호출하세요. 예: `$speckit-specify`, 또는 설치된 bug
확장의 `$speckit-bug-assess`.
다른 통합도 `.agents/skills/`를 대상으로 합니다. 이 디렉터리를 공유하는 다른
에이전트를 설치하기 전에
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을
검토하세요.

## 옵션과 워크플로 디스패치

`--integration-options="--skills"`는 스킬 레이아웃을 명시적으로 선택하는데, 이는
이미 이 통합에서 지원되는 유일한 레이아웃입니다. 네이티브 이벤트는
`.codex/config.toml`을 사용하며, `--events false`는 Spec Kit 이벤트 핸들러를
비활성화합니다.
[런타임 이벤트](cli.md#runtime-events)를 참고하세요.

워크플로 스텝은 `codex exec <prompt>`를 사용하며, 설정된 경우 `--model`을, 구조화된
출력이 요청된 경우 `--json`을 함께 씁니다. Codex 개발 모드 등록은 스킬 파일을
심볼릭 링크하는 대신 직접 씁니다.

<a id="workflow-dispatch-settings"></a>

## 워크플로 디스패치 설정

명령 및 프롬프트 스텝은
[`reasoning_effort`와 `context_size`](../workflow-steps/types.md#reasoning-effort-and-context-size)를
설정할 수 있습니다.
Codex 디스패치는 이를 `--model` 뒤에 설정 오버라이드로 전달합니다.

| 스텝 설정 | Codex CLI 인자 | 값 |
|---|---|---|
| `reasoning_effort` | `-c model_reasoning_effort="<value>"` | 선택된 모델이 지원하는 수준. 예: `low`, `medium`, `high`, `xhigh` |
| `context_size` | `-c model_context_window=<value>` | 토큰 수: 앞자리 0이 없는 양의 정수, 최대 `9223372036854775807` |

Spec Kit은 `reasoning_effort`를 변경 없이 그대로 전달합니다. Codex CLI 0.165.0은
이를 확인하지 않고 모델에 전달합니다. Codex는 전달받은 추론 수준을 표시하므로,
캡처된 출력이 아니라 리터럴이나 워크플로 입력으로 설정하세요. 유효한 토큰 수가
아닌 `context_size`는 Codex가 시작되기 전에 스텝을 실패시킵니다.

`context_size`는 Codex가 모델에 대해 가정하는 컨텍스트 윈도를 재정의합니다.
Codex는 값을 모델의 최댓값으로 제한하며, 모델 기본값보다 낮은 값은 사용할 수 있는
컨텍스트를 줄입니다.
