# 워크플로 표현식

스텝은 `{{ expression }}` 구문으로 입력과 이전 스텝 출력을 참조할 수 있습니다.

| 네임스페이스                   | 설명                                 |
| ------------------------------ | ------------------------------------ |
| `inputs.spec`                  | 워크플로 입력 값                     |
| `steps.specify.output.file`    | 이전 스텝의 출력                     |
| `item`                         | 팬아웃 반복의 현재 항목              |
| `fan_in`                       | 팬인 집계 컨텍스트 |
| `context.run_id`               | 현재 워크플로 실행 ID                |
| `context.workflow_dir`         | 워크플로 소스 디렉터리의 해석된 절대 경로. 문자열로 로드된 워크플로에서는 빈 문자열. |

사용 가능한 필터: `default`, `join`, `contains`, `map`, `from_json`, `to_json`, `upper`, `lower`, `split`, `length`.

## 사용자 정의 필터(워크플로 함수)

프로젝트는 Python 호출 가능 객체를 파이프 필터로 설치할 수 있습니다. 패키지와 CLI의
세부 사항은 [워크플로 함수](functions.md)를 참고하세요.

```yaml
# After installing a package that exports trim:
message: "{{ inputs.message | trim | upper }}"
```

함수는 파이프로 전달된 값과 그 뒤의 명시적 필터 인자를 받습니다. 매니페스트의
`min_args`와 `max_args` 범위는 표현식이 평가될 때 적용됩니다. 감싸진 `{{ ... }}`
표현식의 필터 이름과 인자 수는 워크플로 검증 중에 확인되지 않습니다. 최소 인자가 0인 함수에는 괄호 없는 형식을 쓸 수 있고,
괄호를 쓴 호출에는 인자가 하나 이상 들어갑니다. 함수 이름은 내장 필터 및 설치된
다른 패키지와 평면적인 네임스페이스를 공유합니다. 충돌은 오류이며 내장 필터는
절대 가려지지 않습니다.

함수 패키지에는 실행 가능한 Python이 들어 있으며 워크플로 검증 전에 임포트됩니다.
설치하기 전에 패키지 소스를 검토하세요. 팬아웃이 필터를 동시에 평가할 수 있으므로
핸들러는 상태가 없고 스레드에 안전해야 합니다.
함수 실패는 예외 앞에 함수 이름이 붙은 채 일반적인 표현식 오류 경로를 사용합니다.
반환 값은 별도의 함수 전용 타입 계약이 아니라 기존의 표현식 소비자 규칙을
사용합니다. 다른 워크플로 부수 효과와 마찬가지로, 재개 체크포인트 이전의 중단은
핸들러를 반복시킬 수 있으므로 재개는 최소 한 번(at-least-once)입니다.

## 구문

| 형식 | 예시 |
| --- | --- |
| 점 경로 조회 | `{{ inputs.name }}`, `{{ steps.test.output.exit_code }}` |
| 리터럴 | 문자열, 숫자, `true`, `false`, `none`/`null`, `[1, 2]` 같은 목록 |
| 비교 | `==`, `!=`, `>`, `<`, `>=`, `<=` |
| 불리언 논리 | `and`, `or`, `not` |
| 포함 여부 | `in`, `not in` |
| 필터 | `{{ inputs.tags \| split(',') \| length }}` |

이는 전용 표현식 언어이며, 완전한 Jinja2나 Python이 아닙니다.

## 필터

| 필터     | 예시                                       | 동작                                                                                            |
| -------- | ------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| `default`| `{{ val \| default('fb') }}`               | `None`이나 빈 문자열에 대한 폴백                                                                        |
| `join`   | `{{ list \| join(', ') }}`                 | 목록 요소를 문자열로 결합                                                                          |
| `contains`| `{{ text \| contains('sub') }}`           | 부분 문자열 또는 포함 여부 확인                                                                    |
| `map`    | `{{ list \| map('attr') }}`                | 각 항목에서 속성 추출                                                                              |
| `from_json`| `{{ out \| from_json }}`                 | JSON 문자열을 타입이 있는 값으로 파싱                                                              |
| `to_json`| `{{ obj \| to_json }}`                     | 값을 JSON 문자열로 직렬화 — `from_json`의 역. 매핑 키는 문자열이어야 함 |
| `upper`  | `{{ text \| upper }}`                      | 문자열을 대문자로                                                                                  |
| `lower`  | `{{ text \| lower }}`                      | 문자열을 소문자로                                                                                  |
| `split`  | `{{ csv \| split(',') }}`                  | 문자열을 구분자로 나눠 문자열 목록으로                                                             |
| `length` | `{{ items \| length }}`                    | 목록의 요소 수 또는 문자열의 문자 수                                                               |

`default`는 `None`과 빈 문자열에 대해서만 폴백합니다. 다른 거짓 같은(falsy) 값, 즉 `0`, `false`, `[]`, `{}`는 변경 없이 그대로 통과하므로, `{{ count | default(10) }}`은 개수가 0일 때 여전히 `0`을 냅니다. 여기서 falsy는 비어 있음과 같지 않습니다.

비교적 새로운 필터에 대한 참고 사항:

- **타입은 변환되지 않고 검증됩니다.** `upper`와 `lower`는 문자열만 받고, `split`은 문자열 값과 비어 있지 않은 문자열 구분자를 모두 요구하며, `length`는 목록과 문자열을 받지만 매핑은 거부합니다. 그 밖의 것은 문제를 알려 주는 `ValueError`를 발생시킵니다. 변환은 의도적으로 수행하지 않습니다. 타입 불일치는 거의 항상 워크플로가 잘못된 변수에 연결되었다는 뜻이며, 변환된 결과는 그것을 숨기기 때문입니다. 잘못된 수의 인자가 주어진 필터(`| upper('x')`, 구분자 없는 `| split`, `| split(',', 1)`)는 알려진 필터를 잘못 사용한 것으로 보고되며, 이는 완전히 알 수 없는 필터 이름과 구별됩니다. 인자가 둘 이상인 호출은 단일 표현식으로 평가되지 않고 같은 미지원 형식 오류로 넘어갑니다. 이전부터 있던 필터(`join`, `map`, `contains`)는 더 관대하며 바뀌지 않았습니다. `join`은 지원되지 않는 값을 문자열화하고 `map`/`contains`는 예외를 발생시키는 대신 폴백을 반환합니다.
- **`from_json`은 JSON이 아닌 숫자 상수를 거부합니다.** 따옴표 없는 `NaN`, `Infinity`, `-Infinity`는 객체와 배열 안을 포함해 `from_json`을 지목하는 `ValueError`를 발생시킵니다. `"NaN"` 같은 따옴표로 묶인 문자열은 여전히 유효한 JSON 문자열입니다.
- **`to_json` 출력은 결정적입니다.** 객체 키는 정렬되고 비ASCII 문자는 이스케이프되지 않고 그대로 남으므로, 같은 값은 항상 같은 바이트로 직렬화됩니다. 이는 재현성만 제공합니다. 결과를 셸에 그대로 전달해도 안전해지는 것은 **아닙니다**. [보간은 인용이나 이스케이프를 추가하지 않기](#interpolation-and-shell-safety) 때문입니다. 제한되지 않은 JSON을 `run` 필드에 보간하지 마세요.
- **`to_json`은 유한하지 않은 부동 소수점을 거부합니다.** `NaN`, `Infinity`, `-Infinity`는 따옴표 없는 토큰으로 직렬화되는 대신 `to_json`을 지목하는 `ValueError`를 발생시킵니다. 셋 다 유효한 JSON이 아니므로, 이를 내보내면 표준을 준수하는 하위 파서가 거부해야 하는 문자열을 넘기게 됩니다.
- **`to_json`은 문자열 매핑 키를 요구합니다.** JSON 객체는 문자열 키를 가지므로, 다른 키 타입을 가진 매핑(`{1: "a"}`, `{True: "a"}`)은 대신 키 타입을 알려 주는 `ValueError`를 발생시킵니다. `json.dumps`에 맡기면 정수 키는 `"1"`로 변환되어 기존 `"1"` 키와 충돌하고, 혼합된 키 타입은 "not JSON-serializable"로만 보고되는 순서 `TypeError`로 `sort_keys`에서 실패합니다.
- **필터링한 개수를 비교하려면 괄호가 필요합니다.** 필터 체인은 연산자의 피연산자가 될 수 없으므로, `{{ inputs.items | length > 0 }}`처럼 괄호 없이 뒤따르는 비교는 "ambiguous filter precedence" `ValueError`를 발생시킵니다. 그 결과를 비교하려면 필터링한 피연산자를 괄호로 묶으세요. `{{ (inputs.items | length) > 0 }}`. 최소 둘이라는 임계값이라면:

  ```yaml
  condition: "{{ (inputs.items | length) >= 2 }}"
  ```

  비어 있는지 확인하려면 필터링한 개수의 참/거짓을 직접 사용하세요.

  ```yaml
  condition: "{{ inputs.items | length }}"   # 0 is False, any non-zero count is True
  ```

`and`와 `or`는 `true` 또는 `false`를 반환하며, 왼쪽이 결과를 결정하지 못할 때만 오른쪽을 평가합니다. `and`는 왼쪽이 거짓이면 오른쪽을 건너뛰고, `or`는 왼쪽이 참이면 건너뜁니다. 따라서 조건은 빈 값에서 실패할 필터를 보호할 수 있습니다. `{{ inputs.payload != '' and (inputs.payload | from_json | length) }}`는 비어 있는 선택적 `payload`에 대해 유효하지 않은 JSON으로 실행을 실패시키는 대신 거짓이 됩니다. 오른쪽이 평가될 때는 그 오류가 여전히 실행을 실패시킵니다.

연산자는 느슨한 것부터 강한 것 순으로 `or`, `and`, `not`, 그다음 비교 순으로 결합합니다. 모든 비교(`==`, `!=`, `>`, `<`, `>=`, `<=`, `in`, `not in`)는 한 수준을 공유하며 왼쪽에서 오른쪽으로 평가됩니다. 어느 것도 다른 것보다 우선하지 않습니다. 따라서 `{{ a == b > 0 }}`은 `{{ (a == b) > 0 }}`을 뜻합니다. 비교를 결합하려면 괄호나 `and`/`or`를 사용하세요.

모든 `{{ }}` 블록의 본문은 유효한 표현식이어야 합니다. 산문(`{{ some text }}`), 문자·숫자·`_`·`-` 외의 문자가 있는 키(`{{ inputs.weird:key }}`), 산술(`{{ count + 1 }}`)처럼 표현식이 아닌 텍스트는 문제의 토큰을 알려 주는 `ValueError`를 발생시킵니다. 숫자는 `5`, `-5`, `+5`, `1_000`, `0.5`, `.5`, `5.`, `1.5e3`으로 쓸 수 있습니다.

> **호환성:** 이전 릴리스는 키가 존재하면 `{{ inputs.weird:key }}` 같은 표현식을 해석할 수 있었습니다. 이 표현식은 이제 런타임에 실패합니다. 여러분이 통제하는 키의 이름을 바꾸고(예: `weird:key`를 `weird_key`로) 그 참조를 갱신하세요. 따옴표로 묶은 키 조회는 지원되지 않습니다. 워크플로 검증은 실행 전에 이런 표현식을 감지하지 못할 수 있습니다.

예시:

```yaml
condition: "{{ steps.test.output.exit_code == 0 }}"
args: "{{ inputs.spec }}"
message: "{{ status | default('pending') }}"
tag_count: "{{ inputs.tags | split(',') | length }}"
shell_flag: "{{ inputs.branch | upper }}"
```

<a id="interpolation-and-shell-safety"></a>

## 보간과 셸 안전성

표현식은 **단순 문자열 치환**으로 해석됩니다. `{{ ... }}`의 값은 인용이나 이스케이프가 추가되지 않은 채 주변 텍스트에 있는 그대로 끼워 넣어집니다. `args`와 `message` 문자열을 만드는 데는 편리하지만, `shell` 스텝에는 중요한 결과를 낳습니다. `run` 필드는 시스템 셸(POSIX에서는 `/bin/sh -c`)에 넘겨지므로, 보간된 모든 값은 단순한 데이터가 아니라 **셸 구문**으로 해석됩니다.

보간된 값에 `;`, `|`, `&`, `$( )`, 백틱, 따옴표 같은 문자가 들어갈 수 있다면 실제로 실행되는 명령을 바꾸거나 확장할 수 있습니다. 이는 값이 워크플로 작성자의 완전한 통제 아래에 있지 않을 때 가장 중요합니다.

- **워크플로 `inputs.*`** — 워크플로를 실행하는 사람이 제공합니다.
- **이전 스텝의 출력**, 예: `{{ steps.plan.output.stdout }}` — [`capture: true`](../workflow-steps/types.md#capturing-command-and-prompt-output)를 쓴 `command` 또는 `prompt` 스텝의 경우 이는 **AI 에이전트가 만든 텍스트**이며, 그것은 다시 에이전트가 읽은 파일, 티켓, 웹 콘텐츠의 영향을 받을 수 있습니다. 에이전트 출력이 `shell` 스텝으로 흘러 들어갈 때는 신뢰할 수 없는 것으로 취급하세요.

표현식 언어에는 **셸 이스케이프 필터가 없고**, `shell` 스텝 주위에 **샌드박스도 없습니다**. 동적 값을 셸 소스에서 빼놓으려면 `env` 바인딩과 함께 [`strict: true`](../workflow-steps/types.md#shell-step)를 우선하세요. strict 모드는 데이터를 정제하거나, 명령을 검사하거나, 작성된 스크립트가 데이터를 코드로 해석하는 것을 막지 않습니다. `eval`을 사용하거나, 변수로부터 또 다른 `sh -c` 명령을 만들거나, `BASH_ENV` 같은 셸 시작 변수를 통해 신뢰할 수 없는 값을 전달하지 마세요. Windows에서는 `cmd.exe`의 `%VALUE%` 확장이 셸 구문을 끌어들일 수 있습니다. 신뢰할 수 없는 값은 대신 프로그램 안에서 읽으세요. POSIX에서는 변수 확장을 따옴표로 묶고, 옵션 주입을 막기 위해 적절한 곳에서 대상 프로그램의 인자 검증이나 `--`를 사용하세요.

호환성을 위해 `strict`가 생략되었거나 `false`이면 원시 보간이 활성화된 채로 남습니다. 그 주입 위험은 바뀌지 않습니다. 제한할 수 없는 값은 `run` 필드에서 완전히 빼세요. 여러분이 통제하지 않는 값을 보간하는 strict가 아닌 모든 `run` 필드를 면밀히 살피고, 최소한 다음을 지키세요.

- **`enum`/허용 목록으로 원천에서 값을 제한하세요.** `inputs.*`가 strict가 아닌 `run` 필드에 들어간다면, 호출자가 임의의 셸 텍스트를 아예 제공할 수 없도록 알려진 안전한 값의 고정 집합으로 제한하세요.

  ```yaml
  inputs:
    target:
      type: string
      enum: [staging, production]   # caller cannot inject arbitrary text
  ```

- **제한되지 않은 값은 `run`에서 빼세요.** 값을 허용 목록으로 제한할 수 없다면(대부분의 에이전트/`prompt` 출력) `run` 필드에 보간하지 마세요. 고정된 조건을 상대로 `if`/`switch`로 분기하거나, 그것으로 만든 셸 명령이 아니라 `command`/`prompt` 스텝에서 처리하세요.
- **인용은 보안 경계가 아닙니다.** 치환을 따옴표로 감싸면(`'{{ inputs.x }}'`) 셸이 *신뢰할 수 있는* 값을 단일 인자로 취급하고 공백에 의한 단어 분리를 피하는 데 도움이 되지만, 값 자체에 대응하는 따옴표 문자가 들어 있으면 여전히 빠져나와 셸 구문을 주입할 수 있습니다. 제한된 값에 대해 정확성을 위해 인용하세요. *제한되지 않은* 치환을 안전하게 만드는 데 인용에 의존하지 마세요.
- **게이트는 다음 스텝을 검사하지 않으며, `message`는 그대로 출력됩니다.** `gate` 스텝은 자신의 `message`/`show_file`만 렌더링합니다. 뒤따르는 명령을 표시하거나, 해석하거나, 정제하지 않으며, 승인이 주입 가능한 보간을 무력화하지 않습니다. 신뢰할 수 없는 원시 데이터를 `message`에 보간하지 **마세요**. 제어 문자 제거 없이 있는 그대로 출력되므로, 에이전트나 호출자의 출력이 승인 프롬프트를 바꾸거나 숨기는 터미널/ANSI 이스케이프를 주입할 수 있습니다. `message`는 신뢰할 수 있고 제한된 텍스트로 유지하고, 신뢰할 수 없는 자료는 대신 `show_file`을 통해 검토용으로 드러내세요. 그 경로와 내용은 표시 전에 제어/ANSI 문자가 제거됩니다.

`shell` 스텝은 설계상 임의 명령 프리미티브입니다. 이 실무들은 노출을 줄이고 *어느* 명령이 실행되는지를 작성자의 통제 아래에 두지만, 완전히 통제하지 못하는 값을 보간하는 위험을 없애지는 않습니다.

## 런타임 컨텍스트

`{{ context.* }}`는 현재 실행에 대해 엔진이 관리하는 런타임 메타데이터를
노출합니다.

| 변수 | 설명 |
|----------|-------------|
| `context.run_id` | 현재 워크플로 실행 id(`workflow run`의 끝에 Spec Kit이 `Run ID:`로 출력하는 것과 같은 값). 자동 생성된 실행은 `uuid4`에서 온 8자 16진수이고, 운영자가 제공한 id는 하이픈이나 밑줄이 있는 임의의 영숫자 문자열일 수 있습니다. 실행 컨텍스트 밖에서는 빈 문자열. |
| `context.workflow_dir` | 워크플로 소스 파일을 담은 디렉터리의 해석된 절대 경로. 파일에서 로드된 워크플로에서는 YAML 파일의 상위 디렉터리이고, ID로 설치된 워크플로에서는 설치 디렉터리의 절대 경로(예: `<project>/.specify/workflows/<id>/`)이며, 문자열로 로드된 워크플로에서는 빈 문자열입니다. 재개 시에는 첫 실행의 원래 소스 디렉터리가 보존됩니다. |

```yaml
# Stamp telemetry events with the run id for cross-system join.
- id: emit-event
  type: shell
  strict: true
  env:
    RUN_ID: "{{ context.run_id }}"
  run: 'printf ''{"run_id":"%s","event":"started"}\n'' "$RUN_ID" >> events.jsonl'

# Per-run scratch directory.
- id: prep-scratch
  type: shell
  strict: true
  env:
    RUN_ID: "{{ context.run_id }}"
  run: 'mkdir -p "/tmp/run-$RUN_ID"'

# Pass run id into a command for artifact metadata.
- id: tag-artifact
  command: speckit.specify
  input:
    args: "{{ context.run_id }}"

# Reference a sibling file shipped alongside the workflow definition.
- id: apply-config
  type: shell
  strict: true
  env:
    SOURCE_DIR: "{{ context.workflow_dir }}"
  run: 'cp "$SOURCE_DIR/defaults.yml" ./config.yml'
```

이 셸 예시들은 POSIX 구문을 사용합니다. 이식 가능한 스크립트에서는 호출되는
프로그램 안에서 환경을 읽으세요.

## 타입이 있는 값

단일 표현식(`{{ expr }}` 단독)은 문자열 변환 없이 값을 반환합니다. 혼합 템플릿(`prefix {{ expr }}`)은 텍스트를 반환하며, 내장된 null은 빈 텍스트가 됩니다. 구조화된 값이 텍스트 경계를 넘어야 할 때는 `to_json`을 사용하세요.

## 함께 보기

- [워크플로 스텝 레퍼런스](../workflow-steps/types.md)
- [셸 환경 바인딩](../workflow-steps/types.md#shell-step-environment-variables)
- [입력 타입](api.md#input-types)
