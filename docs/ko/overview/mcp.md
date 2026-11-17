# MCP 서버

> [!WARNING]
> `specify mcp`는 실험적 기능입니다. MCP 표면이 안정적이라고 선언되기 전까지
> 명령 목록과 도구 계약이 바뀔 수 있습니다.

```bash
specify mcp
```

**stdio 전용**으로 Model Context Protocol 서버를 시작합니다. 이 명령은 HTTP,
SSE, 데몬/서비스 관리, 전송 방식 선택 옵션을 제공하지 않습니다. MCP 호스트가
`specify mcp`를 로컬 하위 프로세스로 실행하도록 설정하세요.

stdout이 MCP 프로토콜 프레임을 전달하므로, 서버는 Spec Kit 배너, Rich 출력,
시작 메시지, 경고, 로그를 그곳에 출력하지 않습니다.

## 도구

서버는 타입이 지정된 입력과 결과를 가진 일급 도구를 노출합니다. 모두 읽기
전용이고 로컬 전용(네트워크 접근 없음)이며 예상치 못한 인자를 거부합니다.

| 도구 | 용도 |
| --- | --- |
| `specify_version` | 설치된 CLI 버전, 런타임, 시스템, 기능 역량을 반환합니다. 인자를 받지 않습니다. |
| `specify_artifact_list` | 프로젝트 아티팩트 목록(`command`, `template`, `script`, `hook`)의 페이지 하나를 반환합니다 |
| `specify_artifact_info` | 아티팩트 하나와 그 전체 합성 스택을 반환합니다 |
| `specify_workflow_list` | 설치된 워크플로와 건너뛴 항목 경고를 레지스트리 순서로 나열합니다 |

아티팩트 도구는 선택적인 절대 경로 `project_directory`를 받습니다. 기본값은
서버가 실행된 디렉터리입니다.

`specify_artifact_list`는 `limit`(1-1000, 기본값 100)과 `cursor`도 받습니다.
결과에는 `rows`, `next_cursor`, `truncated`가 있습니다. `truncated`가 true인
동안에는 `next_cursor`를 `cursor`로 전달해 다음 페이지를 가져오세요.

`specify_artifact_info`는 `identifier`(이름만 또는 `kind:name`)와 선택적인
`kind`(`command`, `template`, `script`, `hook`)를 받습니다.

`specify artifact lookup`은 MCP로 사용할 수 없습니다. 아직 공유 타입 작업이
없으므로 `specify_artifact_lookup` 도구는 등록되지 않습니다.

범용 도구 세 개도 등록되며, 점 표기 명령 `version`만 지원합니다. 그 밖의 모든
명령 이름은 `unavailable_command` 도구 오류로 거부됩니다.

| 도구 | 용도 |
| --- | --- |
| `specify_list_commands` | 범용 도구가 지원하는 CLI 명령을 나열합니다 |
| `specify_describe_command` | 지원되는 점 표기 CLI 명령 하나를 설명합니다 |
| `specify_run_command` | 지원되는 명령 하나를 실제 Specify CLI로 실행합니다 |

서버는 변경 작업, 설치·업데이트 작업, 워크플로 실행, 확인 절차, 접근 등급을
노출하지 않습니다.

## 설치된 워크플로 목록

`specify_workflow_list`는 선택적인 절대 경로 `project_directory`(기본값: 서버
실행 디렉터리)와 `limit`(1–1000, 기본값 100)을 받습니다. 설치된 워크플로
레지스트리만 읽으며, 사용자 정의 스텝을 불러오거나 카탈로그를 가져오거나 파일을
수리하거나 워크플로를 실행하지 않습니다.

결과에는 순서가 있는 `items`와 불리언 `truncated`가 있습니다. 워크플로 항목은
`kind: "workflow"`, `id`, `name`, `version`, null 가능한 `description`,
`enabled`를 가집니다. 형식이 잘못된 개별 레지스트리 항목은 원래 위치에
`code: "invalid_registry_entry"`, `workflow_id`, `message`를 가진
`kind: "warning"` 항목을 만듭니다. limit는 워크플로와 경고를 모두 셉니다.
`truncated: true`는 생략된 항목이 있음을 명시적으로 나타냅니다. 더 가져오려면
`limit`를 최대 1000까지 늘리세요. 이어 가기 커서는 없습니다. CLI는 전체 목록을
그대로 유지합니다.

레지스트리 파일이 없으면 빈 목록이 나옵니다. 손상되었거나 읽을 수 없는 레지스트리
파일은 `workflow_registry_read_failed`를, 안전하지 않은 스토리지는
`unsafe_workflow_storage`를 만듭니다. 그 밖에 예상되는 오류는
`invalid_project_directory`와 `not_a_spec_kit_project`입니다. 오류는 `error`
아래에 `code`, `message`, `details`, `retryable`을 담고 `isError: true`를
가집니다. 예상치 못한 실패는 정제된 `internal_error`를, 유효하지 않은 작업 결과는
`invalid_operation_result`를 사용합니다. 1 MiB를 초과하는 응답은 부분 성공 대신
재시도 가능한 `response_too_large` 오류를 만듭니다. 더 작은 limit로 재시도하세요.
레지스트리 예외 세부 정보는 MCP로 노출되지 않습니다.

범위를 벗어난 `limit` 값은 작업이 실행되기 전에 거부되며 구조화된 콘텐츠 없는
텍스트 전용 입력 검증 오류를 만듭니다. [입력 검증 오류](#input-validation-errors)를
참고하세요.

워크플로 목록 조회는 선택된 프로젝트의 `FileHelper` 정책 소유자를 통해 레지스트리
내용을 읽으며, 읽기 경계에서 Windows 정션과 지원되지 않는 재분석 지점을
거부합니다. 일반적인 워크플로 레지스트리 심볼릭 링크는 여전히 허용되지 않습니다.
다른 워크플로 레지스트리 호출자는 이 도구로 마이그레이션되지 않습니다. 이 검사들은
OS 샌드박스나 경쟁 조건 없는 파일시스템 격리를 제공하지 않습니다. 레지스트리는
항목 limit가 적용되기 전에 여전히 전체가 파싱됩니다.

요청된 항목 limit를 초과하거나, 그 limit를 채우지 않고 잘림을 주장하는 작업 결과는
`invalid_operation_result`로 거부됩니다. 출력 스키마도 `items`를 1000개로
제한합니다.

<a id="input-validation-errors"></a>

## 입력 검증 오류

일급 도구는 실행 전에 인자를 입력 스키마에 비추어 검증합니다. 예상치 못한 인자,
범위를 벗어난 `limit`, 형식이 잘못된 `cursor`, 지원되지 않는 `kind`는 구조화된
콘텐츠 없는 텍스트 전용 MCP 도구 오류로 거부되므로, 클라이언트는 거기서
`error.code`를 읽을 수 없습니다.

## 버전 도구 결과와 오류

`specify_version`은 결과 객체를 직접 반환합니다. 작업이 실패하면 구조화된
콘텐츠에 `error.code`(`invalid_operation_result` 또는 `internal_error`),
`error.message`, `error.details`를 가진 MCP 도구 오류를 반환합니다.

## 아티팩트 결과와 오류

아티팩트 도구는 결과를 구조화된 콘텐츠로 반환합니다. 작업 실패는 구조화된
콘텐츠에 `error.code`, `error.message`, `error.details`, `error.retryable`을
가진 MCP 도구 오류입니다. 두 아티팩트 도구 모두 `invalid_project_directory`,
`not_a_spec_kit_project`, `artifact_resolution_failed`,
`invalid_operation_result`, `internal_error`를 반환할 수 있습니다.
`specify_artifact_info`는 `invalid_artifact_identifier`, `unknown_artifact`,
`unknown_hook`, `ambiguous_artifact`도 반환할 수 있습니다. 1 MiB보다 큰 결과는
재시도 가능한 `response_too_large` 오류를 반환합니다. list 도구라면 더 작은
`limit`로 재시도하세요.

## 범용 도구 결과와 오류

`specify_run_command`는 실행 중인 CLI와 같은 Python 환경을 사용하는 격리된 자식
프로세스에서 아래의 정식 명령을 호출합니다.

```bash
specify version --json
```

성공하면 MCP 도구는 `cli_version`, `runtime`, `system`, `features`를 담은 그
명령의 JSON 객체를 직접 반환합니다. 결과를 범용 `schema_version`, `ok`,
`result` 필드로 감싸지 않습니다.

CLI가 실패하면 어댑터는 stderr의 구조화된 JSON 오류를 읽어 `error.code`,
`error.message`, `error.details`를 담은 MCP 도구 오류를 반환합니다. 비어 있거나,
형식이 잘못되었거나, 뒤섞였거나, JSON이 아닌 자식 프로세스 출력은 정제된
`adapter_internal_error`로 보고됩니다. 서버는 성공 형태의 폴백으로 대체하거나
트레이스백을 노출하지 않습니다.
