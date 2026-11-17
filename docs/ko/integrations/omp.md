# Oh My Pi

설정하기 전에 [Oh My Pi](https://www.npmjs.com/package/@oh-my-pi/pi-coding-agent)를 설치하세요.
통합 키는 `omp`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration omp
```

기존 Spec Kit 프로젝트에서는 `specify integration install omp`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 명령

Markdown 슬래시 명령은 `.omp/commands/` 아래에 설치됩니다.
에이전트의 채팅에서 점 표기로 호출하세요. 예: `/speckit.specify`, 또는 설치된 bug
확장의 `/speckit.bug.assess`.

## 워크플로 디스패치

워크플로 스텝은 `omp --print <prompt>`를 사용합니다. `--print`는 일회성 모드를
활성화하고 프롬프트는 위치 인자입니다. 어댑터는 설정된 `--model`을 전달하고,
구조화된 출력에는 `--output-format`이 아니라 `--mode json`을 사용합니다.
통합별 설정 옵션은 없습니다. 아래의 런타임 옵션을 참고하세요.

## 런타임 옵션

명령 스텝은 `omp --print`에 스텝별 런타임 설정을 전달할 수 있습니다
([스텝별 통합 설정](../workflow-steps/types.md#per-step-integration-configuration) 참고).

| `integration_options` 키  | OMP 플래그   | 값                                                                       |
| ------------------------- | ------------ | ------------------------------------------------------------------------ |
| `profile`                 | `--profile`  | 격리된 OMP 프로필 이름                                                   |
| `thinking`                | `--thinking` | `off`, `minimal`, `low`, `medium`, `high`, `xhigh`, `max`, `auto` 중 하나 |
| `tools`                   | `--tools`    | 선택할 도구의 쉼표 구분 목록(배타적 허용 목록이 아님)                    |

```yaml
- id: plan-with-omp
  type: command
  command: speckit.plan
  integration: omp
  integration_args:
    - "--no-session"
  integration_options:
    profile: work
    thinking: high
    tools: "read,bash"
  model: "gpt-5"
```

`tools`는 OMP가 활성화할 도구를 선택합니다. 배타적 허용 목록이나 보안 경계가
아닙니다. OMP는 목록에 없는, 발견된 사용자 정의 도구와 확장이 등록한 도구를 여전히
활성화할 수 있으므로 `tools: read`가 읽기 전용 접근을 보장하지는 않습니다.

옵션 값은 비어 있지 않은 문자열이어야 합니다. 알 수 없는 키와 지원되지 않는
`thinking` 값은 OMP가 실행되기 전에 실패합니다. 모델은 스텝의 최상위 `model`
필드로 선택하세요. `integration_options.model`과 `integration_args`의 `--model`
플래그는 모두 거부됩니다.

`integration_args`는 원시 플래그로 OMP에 전달되지만, OMP의 파서는 일부 토큰이
생성된 옵션과 프롬프트를 소비하거나 재배열하게 하므로 각 값은 다음 중 하나여야
합니다.

- `--no-session`, `--no-title`, `--yolo` 같은 값 없는 알려진 플래그, 또는
- `--system-prompt=...`나 `--resume=<id>` 같은 자기 완결적인 `--name=value` 토큰.

값을 받는 단독 플래그(`--system-prompt`, `--resume`), `--` 종결자와 `--=`
형식(OMP가 `--`로 정규화함), 위치 텍스트, `@file` 참조, 짧은 플래그, 그리고
`--help`/`--version`/`--print`/`--export`/`--alias`/`--mode`는 OMP가 실행되기 전에
실패합니다.

인자는 다음 순서로 `omp --print`에 전달되므로, 명령줄에서 뒤의 소스가 앞의 소스
다음에 옵니다.

1. `SPECKIT_INTEGRATION_OMP_EXTRA_ARGS`
2. `integration_args`
3. `integration_options` 플래그
4. 스텝의 `model` 필드에서 온 `--model`
5. `--mode json`(JSON 출력이 꺼져 있으면 `--mode text`), 그다음 프롬프트
