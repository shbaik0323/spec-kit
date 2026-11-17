# 내장 워크플로 스텝 타입

스텝 설정은 `type`으로 실행 동작을 선택합니다. `type`을 생략하면 `command`가 선택됩니다. 스텝 ID는 로컬 표현식 별칭이며, 생성되는 실행 건 ID용으로 예약된 `:`를 포함해서는 안 됩니다.

## 스텝 타입

| 타입         | 용도                                             |
| ------------ | ------------------------------------------------ |
| `command`    | Spec Kit 명령 호출(예: `speckit.plan`)           |
| `prompt`     | AI 코딩 에이전트에 임의의 프롬프트 전송          |
| `shell`      | 셸 명령을 실행하고 출력 캡처                     |
| `github`     | 이슈 또는 풀 리퀘스트에 명시적인 레이블 추가      |
| `init`       | 프로젝트 부트스트랩(`specify init`과 같음)       |
| `slot`       | 이름 있는 워크플로 슬롯. 채워지지 않으면 건너뜀  |
| `gate`       | 계속하기 전에 사람의 승인을 위해 일시 정지       |
| `if`         | 조건 분기(then/else)                             |
| `switch`     | 표현식에 따른 다중 분기 디스패치                 |
| `while`      | 조건이 참인 동안 반복                            |
| `do-while`   | 최소 한 번 실행한 다음 조건에 따라 반복          |
| `fan-out`    | 목록의 각 항목에 대해 스텝 디스패치              |
| `fan-in`     | 팬아웃 스텝의 결과 집계                          |
| `workflow`   | 비공개 입력과 선언된 출력으로 설치된 워크플로 호출 |

## 명령 스텝(기본값)

통합 CLI를 통해 설치된 Spec Kit 명령을 이름으로 호출합니다.

```yaml
- id: specify
  command: speckit.specify
  input:
    args: "{{ inputs.spec }}"
  integration: claude        # Optional: override workflow default
  model: "claude-sonnet-4-20250514"   # Optional: override model
  reasoning_effort: "high"   # Optional: requires integration support
```

위치 인자와 이름 있는 런타임 옵션은
[스텝별 통합 설정](#per-step-integration-configuration)을, `reasoning_effort`와
`context_size`는
[추론 수준과 컨텍스트 크기](#reasoning-effort-and-context-size)를 참고하세요.

## 프롬프트 스텝

임의의 인라인 프롬프트를 통합 CLI에 전송합니다(명령 파일 불필요).

```yaml
- id: security-review
  type: prompt
  prompt: "Review {{ inputs.file }} for security vulnerabilities"
  integration: claude
```

<a id="capturing-command-and-prompt-output"></a>

## 명령 및 프롬프트 출력 캡처하기

`command`와 `prompt` 스텝은 통합 CLI의 출력을 터미널로 스트리밍하므로
`steps.<id>.output.stdout`과 `steps.<id>.output.stderr`은 빈 문자열입니다. 대신 그
출력을 스텝 결과에 남겨 이후 스텝이 읽을 수 있게 하려면 `capture: true`를
설정하세요.

```yaml
- id: verdict
  type: prompt
  prompt: "Reply with exactly READY or BLOCKED for {{ inputs.spec }}"
  capture: true
  capture_max_bytes: 4096   # Optional: per-stream limit (default 1048576)
  timeout: 120              # Optional: seconds before the CLI is killed

- id: branch
  type: if
  condition: "{{ steps.verdict.output.stdout | contains('READY') }}"
  then:
    - id: plan
      command: speckit.plan
```

| 필드                | 적용 대상           | 의미                                                                                                |
| ------------------- | ------------------- | --------------------------------------------------------------------------------------------------- |
| `capture`           | `command`, `prompt` | `true`이면 stdout과 stderr을 스트리밍하는 대신 캡처합니다. 리터럴 불리언이어야 하며 기본값은 `false`. |
| `capture_max_bytes` | `command`, `prompt` | 캡처되는 각 스트림의 한도. 양의 정수, 또는 한도 없음을 뜻하는 `-1`. 기본값 `1048576`(1 MiB). `capture: true` 필요. |
| `timeout`           | `command`, `prompt` | CLI가 종료되기까지의 초. `capture: true`이면 그 프로세스 그룹(POSIX) 또는 작업 객체(Windows)가 종료되며, 데몬화하는 헬퍼처럼 분리되지 않은 한 CLI가 시작한 프로세스도 포함됩니다. `command`에서는 `capture: true`일 때만 적용되고(기본값 `600`), 스트리밍하는 `command` 스텝에는 타임아웃이 없습니다. `prompt`에서는 항상 적용됩니다(기본값 `300`). |

캡처되는 스텝은 실행 중에 아무것도 출력하지 않습니다. 그 출력에는 다음 필드가
추가됩니다.

| 출력 필드          | 의미                                                                       |
| ------------------ | -------------------------------------------------------------------------- |
| `stdout`, `stderr` | 캡처된 텍스트.                                                             |
| `stdout_truncated` | stdout이 `capture_max_bytes`를 초과해 나머지가 버려졌을 때 `true`.          |
| `stderr_truncated` | stderr에 대해 같은 의미.                                                   |
| `timed_out`        | CLI와 그 프로세스가 `timeout`에 종료되었을 때 `true`. `exit_code`는 `-1`.   |
| `interrupted`      | 실행이 중단되었을 때(Ctrl+C) `true`. `exit_code`는 `130`.                   |

**텍스트의 정체.** 캡처는 통합 CLI의 일반 텍스트 출력, 즉 스트리밍 스텝이
출력하는 것과 같은 출력을 요청합니다. 구조화된 형식이 아닙니다. 최종 답변뿐 아니라
진행 상황이나 도구 사용 줄이 포함될 수 있고, 통합마다 다르며, 에이전트가 JSON만
출력했을 때만 유효한 JSON입니다.

**한도와 인코딩.** `capture_max_bytes`는 디코딩 전에 각 스트림의 원시 바이트를
셉니다. 한도를 넘는 바이트는 CLI가 막히지 않도록 읽은 뒤 버려지고, 해당하는
`*_truncated` 필드가 설정됩니다. 출력은 UTF-8로 디코딩됩니다. 유효하지 않은
바이트는 U+FFFD가 되고, 한도·타임아웃·중단으로 잘린 문자는 대체되지 않고
버려집니다. 줄 끝은 `\n`으로 정규화됩니다.

`capture_max_bytes: -1`이면 아무것도 버려지지 않고 두 `*_truncated` 필드는 모두
`false`로 남습니다. 이 경우 전체 출력이 메모리에 유지되고 실행 상태에 기록되므로,
대량으로 출력하는 에이전트는 둘 다 한없이 키웁니다. 작다고 알고 있는 출력에만
사용하세요.

**출력을 사용하기 전에 완전한지 확인하세요.** `from_json` 같은 필터는 이 필드들을
보지 않고, 거기 있는 텍스트를 그대로 파싱합니다. 출력은 `stdout_truncated`가
`true`일 때, 그리고 `timed_out`이나 `interrupted`가 `true`일 때도 불완전합니다.
멈춘 CLI는 크기 한도가 없어도 부분 출력을 남깁니다. 사용을 방어하세요.

```yaml
- id: parse
  type: if
  condition: "{{ steps.verdict.output.exit_code == 0 and not steps.verdict.output.stdout_truncated }}"
  then:
    - id: use
      type: shell
      run: "echo parsed"
```

**실패.** 0이 아닌 종료, 타임아웃, 중단은 스텝을 실패시키고 캡처된 것을
`output.stdout`과 `output.stderr`에 유지합니다. 스텝의 오류는 짧은 요약이며
(`Command exited with code 2`, `Prompt timed out after 120 seconds.`), 캡처된
텍스트가 아닙니다. 실패한 스텝은 실행을 멈추므로, 이후 스텝이 실패를 살펴봐야
한다면 캡처하는 스텝에 `continue_on_error: true`를 설정하세요.

```yaml
- id: verdict
  type: prompt
  prompt: "Reply with exactly READY or BLOCKED"
  capture: true
  timeout: 120
  continue_on_error: true

- id: on-timeout
  type: if
  condition: "{{ steps.verdict.output.timed_out }}"
  then:
    - id: review
      type: gate
      message: "The verdict step timed out. Approve to continue without it."
```

> ⚠️ **캡처된 출력은 일반 텍스트로 저장되며 가려지지 않습니다.**
> `specify workflow resume`이 재생할 수 있도록 실행의 `state.json`에 기록되고,
> 실행 후에도 그곳에 남습니다. 에이전트가 파일이나 환경에서 읽은 시크릿을 포함해
> 출력한 모든 것이 영속화됩니다. 캡처된 텍스트는 터미널에 출력되지 않으며
> `log.jsonl`에 기록되지 않습니다. 실행 디렉터리가 그에 맞게 보호되지 않는 한,
> 출력에 자격 증명이 들어갈 수 있는 스텝에서는 캡처를 활성화하지 마세요.

캡처된 텍스트는 에이전트 출력이므로 신뢰할 수 없습니다. `shell` 스텝에서 사용하기
전에
[보간과 셸 안전성](../workflows/expressions.md#interpolation-and-shell-safety)을
참고하세요.

<a id="per-step-integration-configuration"></a>

## 스텝별 통합 설정

명령 스텝은 이를 지원하는 통합에 구조화된 런타임 설정을 전달할 수 있습니다.

```yaml
- id: implement-with-docker-agent
  type: command
  command: speckit.implement
  integration: docker-agent
  integration_args:
    - "{{ inputs.agent_config }}"
  integration_options:
    agent: root
    safety: balanced
  model: "openai/gpt-5"
  input:
    args: "{{ inputs.spec }}"
```

`integration_args`는 순서가 있는 문자열 목록입니다. `integration_options`는 문자열
키를 가진 매핑입니다. 두 필드의 값은 워크플로 표현식 메커니즘으로 해석되고 선택된
통합이 검증합니다. 지원되지 않거나, 알 수 없거나, 형식이 잘못된 값은 조치 가능한
오류와 함께 실패합니다. Docker Agent는 단일 위치 인자를 에이전트 설정 참조로
사용하고, 이름 있는 통합 옵션으로 `agent`와 `safety`를 받습니다. 모델은 명령
스텝의 최상위 `model` 필드로 설정하세요.

Docker Agent의 `integration_args`가 명령 스텝에 에이전트 참조를 제공하면 그것이
`SPECKIT_INTEGRATION_DOCKER_AGENT_EXTRA_ARGS`보다 우선합니다. 그 스텝에서는 레거시
환경 값 전체가 무시됩니다. 스텝별 에이전트 참조가 없으면 레거시 환경 동작은
바뀌지 않습니다.

해석된 런타임 설정은 워크플로 실행 상태에 기록됩니다. 실패했거나 일시 정지된
명령이 재개되면 전체 디스패치 설정이 현재 입력으로부터 다시 해석되므로,
`workflow resume --input`으로 제공한 값이 일관되게 적용됩니다. 입력을 갱신하지
않은 재개는 같은 값을 재현합니다.

`workflow.options` 위에 병합되는 명령 스텝의 `options` 매핑은
`steps.<id>.output.options`로 기록되지만 통합 CLI에는 전달되지 않습니다. 호출을
바꾸려면 `integration_args`, `integration_options`, `model`, 또는
[`reasoning_effort`와 `context_size`](#reasoning-effort-and-context-size)를
사용하세요.

<a id="reasoning-effort-and-context-size"></a>

## 추론 수준과 컨텍스트 크기

`command`와 `prompt` 스텝은 자신이 호출하는 통합 CLI에 대해 `reasoning_effort`와
`context_size`를 설정할 수 있습니다. `workflow.reasoning_effort`와
`workflow.context_size`는 두 스텝 타입 모두의 기본값을 설정합니다.

```yaml
workflow:
  id: plan-and-review
  name: Plan and review
  reasoning_effort: "medium"   # Default for command and prompt steps

inputs:
  review_effort:
    type: string
    default: ""                # Empty: the review step sends no effort

steps:
  - id: plan
    command: speckit.plan
    integration: codex         # Codex takes context_size as a token count
    reasoning_effort: "high"   # Overrides the workflow default
    context_size: "200000"     # Quoted: values are strings

  - id: review
    type: prompt
    prompt: "Review the plan for gaps"
    reasoning_effort: "{{ inputs.review_effort }}"
```

**해석.** 각 설정은 `model`처럼 해석됩니다. 스텝의 값, 그다음 워크플로 기본값,
그다음 미설정입니다. 스텝 값과 기본값 모두 표현식일 수 있습니다. 스텝 값이 없거나,
`null`이거나, `""`이면 기본값을 상속합니다. `""`나 null로 해석되는 표현식은
기본값으로 폴백하지 않고 그 스텝의 설정을 해제합니다. 스텝이 기본값에서 빠지는
방법이 이것입니다. 예시에서 `review`는 실행 시
`--input review_effort=<value>`를 제공하지 않는 한 추론 수준을 보내지 않습니다.

**값.** 값은 문자열입니다. 숫자와, YAML이 불리언으로 읽는 `on`, `off`, `yes`, `no`
같은 단어는 따옴표로 묶으세요. `context_size: 200000`이 아니라
`context_size: "200000"`으로 쓰세요. 표현식도 문자열로 해석되어야 하므로, 숫자는
`type: number`가 아니라 `type: string`으로 선언된 입력에서 읽으세요.
값은 문자나 숫자로 시작하는 1~64자의 문자, 숫자, `.`, `_`, `-`여야 합니다. 리터럴
값은 워크플로가 검증될 때, 표현식 결과는 스텝이 실행될 때 확인됩니다. 형식이
잘못된 값에 대한 오류는 그 텍스트가 아니라 타입과 길이만 보고합니다. Spec Kit은
값을 변경 없이 통합에 전달합니다. 그 의미와 허용되는 값은 통합의 CLI가 정의하며,
통합은 CLI가 실행되기 전에 값을 거부할 수 있습니다.

**지원.** 통합은 지원을 선언한 설정만 변환합니다. 그렇지 않으면 스텝은 CLI가
실행되기 전에 실패하며, 오류는 설정, 그 출처(스텝 또는 워크플로 기본값), 그것을
지원하는 통합을 알려 줍니다. 워크플로 기본값도 예외가 아닙니다. 기본값은 통합이
그것을 지원하지 않는 모든 상속 스텝을 실패시킵니다. 다음 내장 통합이 이 설정을
지원합니다.

| 통합 | `reasoning_effort` | `context_size` |
| --- | --- | --- |
| [GitHub Copilot](../integrations/copilot.md#workflow-dispatch-settings) | 예 | 예 |
| [Codex CLI](../integrations/codex.md#workflow-dispatch-settings) | 예 | 예 |
| [Claude Code](../integrations/claude.md#workflow-dispatch-settings) | 예 | 아니요 |

다른 내장 통합은 어느 설정도 지원하지 않습니다. 각 통합의 가이드가 CLI 인자와
허용되는 값을 설명합니다. 통합이 참여를 선언하는 방법은
[워크플로 CLI 디스패치](../integrations/cli.md#workflow-cli-dispatch)를
참고하세요.

**CLI의 거부.** CLI가 값을 거부하고 0이 아닌 코드로 종료하면 스텝은 다른 CLI
실패와 마찬가지로 실패합니다. 스텝 오류는 종료 코드를 보고하고, CLI의 메시지는
터미널에, 또는 [`capture: true`](#capturing-command-and-prompt-output)일 때는
`output.stderr`에 나타납니다.

**기록과 재개.** 해석된 값은 `steps.<id>.output.reasoning_effort`와
`steps.<id>.output.context_size`에 기록되며, 설정되지 않으면 `null`입니다.
`specify workflow resume`이 스텝을 다시 실행할 때는 현재 입력으로부터 둘 다 다시
해석하므로 `--input`으로 제공한 값이 적용됩니다.

**추가 인자.** 이 설정을 지원하는 통합은 그 플래그를
`SPECKIT_INTEGRATION_<KEY>_EXTRA_ARGS`의 인자 뒤에 추가합니다. 둘이 같은 CLI
플래그를 설정하면 어느 값이 이길지는 CLI가 결정하므로, 각 플래그는 한 곳에서만
설정하세요.

**호환성.** 이 기능이 없는 Spec Kit 버전은 워크플로 기본값을 포함해 이 필드들을
무시하고 그것 없이 스텝을 실행합니다.
`requires.speckit_version`은 이를 막지 못합니다. 권고 사항일 뿐 강제되지 않습니다.

<a id="shell-step"></a>

## 셸 스텝

`type: shell`은 플랫폼 셸을 통해 `run`을 실행하며, 파이프, 리디렉션, 여러 명령에
대한 지원을 유지합니다. 따옴표 안을 포함해 `run` 어디에서든 `{{` 표현식 마커를
거부하려면 `strict: true`를 설정하세요. 이는 검증 중과 실행 직전에 모두
확인됩니다. `strict`는 불리언이어야 합니다. 생략하거나 `false`이면 기존의 원시
보간 동작이 유지됩니다.

동적 값을 셸 소스에 삽입하지 않고 바인딩하려면 `env` 매핑을 사용하세요. 예를 들어
POSIX에서:

```yaml
- id: display-result
  type: shell
  strict: true
  env:
    RESULT: "{{ steps.plan.output.stdout }}"
  run: 'printf "%s\n" "$RESULT"'
```

크로스 플랫폼 스크립트에서는 Python의 `os.environ["RESULT"]`처럼 프로그램
자체에서 환경을 읽으세요. 바인딩 계약은
[셸 스텝 환경 변수](#shell-step-environment-variables)를, 남아 있는 신뢰 경계는
[보간과 셸 안전성](../workflows/expressions.md#interpolation-and-shell-safety)을
참고하세요.

> **보안 참고:** `shell` 스텝은 **여러분의** 권한으로 로컬 명령을 실행합니다. 기능 샌드박스는 없습니다. `requires`는 권고용 사전 조건 블록(spec-kit 버전, 통합)이지 런타임 게이트가 아니므로, 스텝이 할 수 있는 일을 제한하지 **않습니다**. 특히 `requires.permissions` 기능 게이트는 없습니다. 존재하지 않는 샌드박스를 암시하게 되므로 바로 그 이유로 검증에서 거부됩니다. 카탈로그나 다운로드한 워크플로는 실행하기 전에 검토하고, 민감하거나 파괴적인 셸 명령 전에 명시적인 승인을 요구하려면 `gate` 스텝을 사용하세요.

셸 `timeout`은 양수여야 하며 기본값은 `300`초입니다. 타임아웃은 명령을 종료하고 스텝을 실패시킵니다. 셸 출력은 이후의 표현식을 위해 캡처됩니다.

stdout을 타입이 있는 `output.data`로 파싱하려면 `output_format: json`을
설정하세요. 따옴표 없는 `NaN`, `Infinity`, `-Infinity` 상수를 포함해 유효하지 않은
JSON은 `stdout`, `stderr`, `exit_code`를 보존한 채 스텝을 실패시킵니다. `"NaN"`
같은 따옴표로 묶인 문자열은 여전히 유효합니다. `output_format`이 없으면 stdout은
`data` 필드 없이 원시 텍스트로 남습니다.

<a id="shell-step-environment-variables"></a>

## 셸 스텝 환경 변수

셸 스텝은 다음 환경 변수를 자동으로 받습니다.

| 변수 | 설명 |
| -------- | ----------- |
| `SPECKIT_WORKFLOW_DIR` | 워크플로 소스 디렉터리의 해석된 절대 경로(`{{ context.workflow_dir }}`와 같은 값). 워크플로에 소스 경로가 없으면 설정되지 않습니다. |

선택적인 스텝별 `env` 매핑은 strict 모드와 비strict 모드 모두에서 사용할 수
있습니다. 이름은 `[A-Za-z_][A-Za-z0-9_]*`와 일치해야 하며, 이름은 표현식이 아니라
리터럴입니다. 값은 문자열이어야 하고 워크플로 표현식을 포함할 수 있습니다.
각 값은 한 번 평가되며 NUL 문자가 없는 문자열로 해석되어야 합니다. POSIX에서는 그
문자열이 파일시스템 인코딩과 그 오류 처리기를 사용해 프로세스 환경용으로 인코딩될
수도 있어야 합니다.
유효하지 않은 바인딩은 프로세스를 실행하기 전에 스텝을 실패시킵니다.

단일 표현식 바인딩은 표현식의 결과 타입을 보존합니다.
`COUNT: "{{ inputs.count }}"`는 숫자, 불리언, null, 목록, 매핑을 텍스트로 변환하는
대신 거부합니다. 혼합 템플릿은 표준 표현식 보간을 사용합니다.
`COUNT: "n={{ inputs.count }}"`는 정수 `42`에 대해 `n=42`를 만듭니다. 내장된
null은 빈 텍스트가 되고, 다른 값은 그 문자열 표현을 사용합니다. 구조화된 데이터를
명시적으로 직렬화하려면 `to_json`을 사용하세요.

바인딩은 그 하위 프로세스에 대해서만 상속된 환경 값을 재정의합니다. 부모
프로세스나 이후 스텝을 수정하지 않습니다. 빈 문자열은 유효합니다.
`SPECKIT_WORKFLOW_DIR`은 (대소문자 구분 없이) 예약되어 있으며 재정의할 수
없습니다. 환경 이름과 실행 파일 로딩 설정은 워크플로 작성자의 통제 아래에 두세요.
환경 바인딩은 데이터 채널이지 샌드박스가 아닙니다.

<a id="reviewing-shell-execution"></a>

### 셸 실행 검토하기

낯선 워크플로를 설치하거나 실행하기 전에, 중첩된 분기와 팬아웃 템플릿을 포함해 각
셸 스텝의 `run`**과** `env`를 감사하세요. 참조된 스크립트와 리소스, 호출되는
워크플로, 스텝을 교체하거나 삽입할 수 있는 활성 오버레이 YAML을 검토하세요. 번들의
워크플로 컴포넌트에도 같은 검토를 적용하고, 정의, 오버레이, 참조된 코드가 바뀌면
다시 검토하세요.
사용자 정의 스텝 패키지는 로드 시 Python 코드를 실행하므로 자체 소스 검토가
필요합니다.

환경 값은 단순히 데이터를 제공하는 것이 아니라 실행을 제어할 수 있습니다.
예를 들어 `PATH`는 실행 파일을 선택하고, `PYTHONPATH`는 Python 임포트에 영향을
주고, `BASH_ENV`는 Bash 시작 코드를 선택할 수 있고, `NODE_OPTIONS`는 Node.js 시작
동작을 바꿀 수 있습니다. 네이티브 로더와 다른 프로그램에는 각자의 설정이 있습니다.
이 예시들은 완전한 목록이 아닙니다. 따라서 고정된 `run`도 `strict: true`이고
`eval`이 없더라도 다른 코드를 실행할 수 있습니다. strict 모드는 실행을 제어하는
바인딩을 걸러 내지도, 상속된 환경을 정리하지도 않습니다.
그런 설정은 리터럴로, 작성자가 검증한 상태로 유지하거나, 동적 값을 승인된 신뢰
경로/옵션 집합으로 제한하세요. 제한 없는 입력이나 에이전트 출력은 소비자가 그것을
데이터로 다루는 애플리케이션 데이터 변수에만 사용하세요.

셸 스텝이 Specify 자체를 호출할 때는 `SPECKIT_WORKFLOW_CATALOG_URL` 같은 카탈로그
오버라이드가 하위 명령이 컴포넌트를 탐색하거나 설치하는 위치도 바꿀 수 있습니다.
그런 바인딩은
[활성 카탈로그 소스](../overview/catalogs.md#discovery-and-trust)와 함께
검증하세요.

워크플로 검증은 설정을 확인할 뿐, 환경 설정이나 명령이 안전한지는 확인하지
않습니다. `specify workflow info`는 메타데이터와 스텝 정체성을 보여 주고,
`specify workflow resolve`는 레이어와 스텝 귀속을 보여 줍니다. 어느 쪽도 전체
`run`/`env` 설정을 표시하지 않으며 기여하는 소스 파일을 읽는 일을 대체하지
않습니다. 카탈로그 설치 권한, 번들 설치, strict 모드는 보안 보증이 아닙니다.

## GitHub 스텝

`type: github`는 설치된 `gh` CLI와 그 활성 인증을 사용합니다.
GitHub 저장소를 `owner/repo`로 지정하세요. Git 원격이나 소스 체크아웃은 필요하지
않습니다. 요청은 그 저장소에 대해 GitHub의 `api.github.com` REST 엔드포인트로
갑니다. 활성 토큰에는 이슈나 풀 리퀘스트를 읽고 그 레이블을 쓸 권한이
필요합니다(예: Actions에서 `GITHUB_TOKEN`을 사용할 때 이슈에는 `issues: write`,
PR에는 `pull-requests: write`). 스텝은 기존 토큰만 사용합니다. 권한을 요청하거나
상승시키지 않습니다. 이슈 내용에서 지침을 찾거나 메인테이너의 결정을 내리지
않습니다.

```yaml
- id: mark-ready
  type: github
  operation: add-label
  repository: owner/repo
  target: issue
  number: "{{ inputs.issue_number }}"
  label: ready-for-review
```

`target`은 `issue` 또는 `pull_request`입니다. `number`는 양의 정수로 해석되어야
합니다. `label`은 명시적인 레이블 이름 하나(최대 50자)이며 저장소에 이미 존재해야
합니다. `repository`, `number`, `label`은 워크플로 표현식을 사용할 수 있습니다.
저장소는 URL이나 Git 원격이 아니라 단일 `owner/repo` 이름으로 해석되어야 합니다.
스텝은 대상 유형과 기존 레이블을 확인하고, 레이블이 이미 있으면 쓰기를 건너뛰고,
추가한 뒤에는 GitHub의 응답을 검증합니다.
출력에는 `repository`, `target`, `number`, `label`, `added`(레이블이 추가되었을
때만 `true`)가 포함됩니다. 반복 실행과 팬아웃 항목은 안전하게 재시도할 수
있습니다. 이미 있는 레이블은 변경 없이 유지됩니다. 이 첫 버전은 댓글을 게시하거나,
아티팩트를 복원하거나, PR을 체크아웃하거나, 메인테이너가 어떤 레이블을 고를지
추론하지 않습니다.

## Init 스텝

`specify init`과 같은 방식으로 프로젝트를 부트스트랩합니다. 템플릿, 스크립트, 공유
인프라, 선택한 코딩 에이전트 통합을 스캐폴딩합니다. 비대화형으로 실행되고(기본값은
`--ignore-agent-tools`), 스텝 설정이나 워크플로 기본값으로부터 통합을 해석합니다.

```yaml
- id: bootstrap
  type: init
  here: true                 # or: project: my-project
  integration: copilot       # Optional: defaults to workflow integration
  integration_options: "--skills"  # Optional: extra options for the integration
  script: sh                 # Optional: sh, ps, or py
  force: true                # Optional: required when target directory already exists
  preset: healthcare-compliance   # Optional preset ID
```

## 워크플로 슬롯

다운스트림 프로젝트가 워크플로 오버레이로 채울 수 있는 이름 있는 워크플로 슬롯을
선언합니다. 슬롯은 채워지지 않으면 건너뜁니다. 그 `id`는 오버레이 앵커이고
`name`은 사람이 읽을 수 있는 필수 레이블입니다.

```yaml
- id: post-implement
  type: slot
  name: "Post-implementation checks"
```

슬롯 채우기, 그 출력 보존, 팬아웃 제한은
[워크플로 슬롯](../workflows/overlays.md#workflow-slots-upstream-extension-points)을
참고하세요.

## 게이트 스텝

대화형 터미널에서 사람의 검토를 요청하거나, 이후의 `specify workflow resume`을
위해 일시 정지합니다. 판정 입력은 프롬프트 없이 결정할 수 있습니다.

```yaml
- id: review-spec
  type: gate
  message: "Review the generated spec before planning."
  options: [approve, edit, reject]
  on_reject: abort
```

## 게이트 판정 입력

`verdict_input`은 게이트의 판정을 이름 있는 워크플로 입력에 바인딩합니다. 입력은
워크플로의 `inputs` 블록에 선언되어 있어야 합니다. `add`나 `run` 중의 워크플로
검증이 선언되지 않은 참조를 보고합니다.

`verdict_input`은 `fan-out` 템플릿 안에서는 지원되지 않습니다. 팬아웃 항목은
워크플로 입력을 공유하는 반면, 워크플로 상태는 일시 정지된 게이트를 하나만 표현할
수 있습니다. 배치 전체를 승인하려면 팬아웃 앞에, 집계된 결과를 검토하려면 팬인
뒤에 게이트를 두세요.

**입력 값 의미론:**

| 값 | 동작 |
|---|---|
| 비어 있지 않은 문자열, 옵션과 일치(대소문자 구분 없음) | 게이트가 자동 결정. `output.choice`는 설정된 옵션 표기로 설정됨 |
| 비어 있지 않은 문자열, 일치 없음 | 게이트가 즉시 실패 |
| 문자열이 아님 | 게이트가 즉시 실패 |
| 없거나 비어 있음 | TTY에서는 프롬프트 표시. 그렇지 않으면 일시 정지 |

**기본값 의미론:** 비어 있지 않은 `default`는 첫 실행에서 판정으로 소비됩니다. 옵션과 일치하면 게이트가 자동 결정되고, 일치하지 않으면 즉시 실패합니다.

```yaml
inputs:
  spec_verdict:
    type: string
    default: ""
steps:
  - id: review-spec
    type: gate
    message: "Approve the specification?"
    options: [approve, reject]
    on_reject: retry
    verdict_input: spec_verdict
```

재개할 때 판정을 제공합니다.

```bash
specify workflow resume <run_id> --input spec_verdict=approve
```

`on_reject: retry`의 경우, 바인딩된 reject 판정은 게이트가 일시 정지하기 전에
소비됩니다. 이름 있는 저장 입력이 `""`로 재설정됩니다. 따라서 이후의 재개는 다른
판정이 제공될 때까지 다시 프롬프트를 표시하거나 일시 정지합니다. approve, abort,
skip 결과는 입력을 변경하지 않습니다.

그 재설정 때문에 `on_reject: retry`와 함께 쓰는 판정 입력은 `""`를 받아들여야
합니다. `enum`을 선언한다면 빈 문자열을 포함하세요. 그렇지 않으면 재설정 값이
입력 자신의 `enum`을 위반해 어떤 입력으로도 실행을 재개할 수 없게 됩니다.
`specify workflow add`는 이를 검증 오류로 보고합니다.

```yaml
inputs:
  spec_verdict:
    type: string
    enum: ["", approve, reject]
    default: ""
```

## If/Then/Else 스텝

표현식에 기반한 조건 분기입니다.

```yaml
- id: check-scope
  type: if
  condition: "{{ inputs.scope == 'full' }}"
  then:
    - id: full-plan
      command: speckit.plan
  else:
    - id: quick-plan
      command: speckit.plan
      options:
        quick: true
```

## Switch 스텝

표현식 값에 따른 다중 분기 디스패치입니다.

```yaml
- id: route
  type: switch
  expression: "{{ steps.review.output.choice }}"
  cases:
    approve:
      - id: plan
        command: speckit.plan
    reject:
      - id: log
        type: shell
        run: "echo 'Rejected'"
  default:
    - id: fallback
      type: gate
      message: "Unexpected choice"
```

## While 루프 스텝

조건이 참인 동안 스텝을 반복합니다.

```yaml
- id: retry
  type: while
  condition: "{{ steps.run-tests.output.exit_code != 0 }}"
  max_iterations: 5
  steps:
    - id: fix
      command: speckit.implement
```

## Do-While 루프 스텝

스텝을 최소 한 번 실행한 다음 조건이 유지되는 동안 반복합니다.

```yaml
- id: refine
  type: do-while
  condition: "{{ steps.review.output.choice == 'edit' }}"
  max_iterations: 3
  steps:
    - id: revise
      command: speckit.specify
```

## 팬아웃 스텝

컬렉션의 각 항목에 대해 스텝 템플릿을 디스패치합니다. 기본값인
`max_concurrency: 1`은 순차 실행입니다. 더 높은 값은 독립적인 항목 컨텍스트와
순서가 유지되는 결과로 항목을 동시에 실행합니다.

```yaml
- id: parallel-impl
  type: fan-out
  items: "{{ steps.tasks.output.task_list }}"
  max_concurrency: 3
  step:
    id: impl
    command: speckit.implement
```

## 팬인 스텝

팬아웃 스텝의 결과를 집계합니다.

```yaml
- id: collect
  type: fan-in
  wait_for: [parallel-impl]
  output: {}
```

`wait_for`는 선언된 스텝 ID를 참조합니다. 팬아웃의 경우 팬아웃 스텝 자신의
`id`(위의 `parallel-impl`)를 사용하세요. 순서가 유지되는 항목 결과는
`steps.<fan-out-id>.output.results`로 사용할 수 있습니다. `fan:template:0` 같은
생성된 항목 별칭은 보고 전용이며 유효한 `wait_for` 대상이 아닙니다.

<a id="workflow-composition"></a>

## 워크플로 합성

`workflow` 스텝은 현재 프로젝트에 설치되고 활성화된 워크플로를 같은 실행의 비공개
범위로 실행합니다. 대상은 리터럴 ID 또는 표현식일 수 있습니다. 해석된 문자열은
대소문자와 공백을 포함해 ID와 정확히 일치해야 합니다.

```yaml
inputs:
  target: {type: string, required: true}
  report: {type: string, required: true}
steps:
  - id: investigate
    type: workflow
    workflow: "{{ inputs.target }}"
    input:
      report: "{{ inputs.report }}"
```

포함된 워크플로는 `input`을 통해 전달된 선언된 입력과 자신의 스텝 결과만 봅니다.
호출자의 `inputs`, 스텝 결과, 팬아웃 `item`/`fan_in` 값, 워크플로 기본값을
상속하지 않습니다. 알 수 없는 입력 이름, 누락된 필수 값, 유효하지 않은 타입/enum은
호출을 실패시킵니다. 기존의 하위 기본값과 `integration: auto` 해석이 적용됩니다.
값은 명시적인 선언을 통해서만 되돌아옵니다.

```yaml
outputs:
  report:
    value: "{{ steps.analyze.output.stdout }}"
```

호출자는 `{{ steps.investigate.output.report }}`를 읽습니다. 출력에는 `workflow`와
`status`가 포함됩니다. 실패에는 실패한 작업이 보고한 경우 `error`가, 중단에는
`aborted: true`가 포함됩니다. 이 이름들과 `integration`, `model`, `options`,
`input`은 예약되어 있습니다. 반환되는 값은 JSON으로 안전해야 합니다. 비공개 입력,
스텝 기록, 로그는 반환 값의 일부가 아닙니다. 별도의 하위 실행은 만들어지지
않습니다.

워크플로 호출의 `continue_on_error: true`는 반환된 하위 실패와, 사용할 수 없는
대상, 유효하지 않은 매핑 입력, 순환, 깊이 한도, 유효하지 않은 선언 출력 같은 호출
경계 계약 실패를 처리합니다. 스텝 예외, 표현식 오류(`input:`이나 `outputs:`의
`from_json` 오류 포함), 중단, 체크포인트 실패는 잡지 않습니다. 이들은 루트에서와
똑같이 전파됩니다. 사용할 수 없는 스텝 구현은 그 스텝이 나타나는 곳에서
종결적입니다. 워크플로 호출에서는 대상을 바인딩하는 중에 감지되든 나중에 감지되든
보고되는 하위 실패입니다. 일시 정지와 명시적 중단은 항상 실행을 멈춥니다.

중첩 합성은 허용되지만, 활성 호출 경로에서 반복되는 워크플로 ID는 순환입니다.
다이아몬드 구조는 허용됩니다. 포함의 최대 깊이는 16이며 루트가 깊이 0입니다.

## 하위 게이트 호출하기

`workflow resume --input`에는 루트에 선언된 입력만 제공할 수 있습니다. 하위
게이트가 루트 입력을 `verdict_input`으로 사용한다면 각 워크플로 경계를 거쳐 그
입력을 매핑하세요.

```yaml
# Parent workflow
inputs:
  approval:
    type: string
    default: ""
steps:
  - id: review-release
    type: workflow
    workflow: release-notes
    input:
      approval: "{{ inputs.approval }}"
```

```yaml
# Installed release-notes workflow
inputs:
  approval:
    type: string
    default: ""
steps:
  - id: review
    type: gate
    message: "Approve the release notes?"
    options: [approve, reject]
    verdict_input: approval
```

`approval`이 비어 있으면 최초 실행은 일시 정지합니다. 그 구조화된 상태는
`gate.scope_path`로 중첩된 게이트와 그것을 감싸는 호출을 식별합니다. 계속하려면
하위가 아니라 루트 실행을 재개하세요.

```bash
specify workflow status <run-id> --json
specify workflow resume <run-id> --input approval=approve
```

`verdict_input`을 가진 게이트는 하나 이상의 워크플로 호출을 거치는 경우를 포함해
팬아웃 항목 안에서는 지원되지 않습니다. `inside_fan_out` 런타임 조건은 워크플로
경계를 넘어 보존됩니다.

## 호출 경계에서의 기본값과 오류

상위의 `integration`, `model`, `options`, `reasoning_effort`, `context_size`
기본값은 상위 워크플로의 스텝에만 적용됩니다. 하위는 자신의 기본값이나 자동 해석을
사용합니다. 값을 둘 다에 공통으로 만들려면 하위 입력으로 선언하고 명시적으로
매핑하세요.
호출 결과는 의도적으로 빈 `input` 필드를 가지며 하위의 비공개 입력, 스텝 기록,
로그를 노출하지 않습니다.

최초 바인딩 또는 출력 확정 계약 실패는 `continue_on_error`로 처리할 수 있습니다.
`workflow resume --input` 중에 완료되지 않은 호출을 다시 바인딩하다 발생한 실패는
다릅니다. 그것은 전파됩니다. 이전에 바인딩된 호출의 경우, 재개는 바인딩과 하위
서브트리를 보존하면서 이전의 `result`, `outcome`, `error`를 버리므로 수정된 루트
입력으로 재시도할 수 있습니다. 최초 바인딩이 실패한 호출은 `ready`로 돌아가 대상
해석과 바인딩을 재시도합니다.

## 함께 보기

- [워크플로 스텝 개요](overview.md)
- [워크플로 YAML과 실행 의미론](../workflows/api.md)
- [표현식](../workflows/expressions.md)
- [오버레이와 워크플로 슬롯](../workflows/overlays.md)
- [사용자 정의 스텝 패키지 개발하기](developing.md#custom-step-packages)
- [스텝 관리 CLI](cli.md#step-management)
