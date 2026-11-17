# 워크플로 스텝 API 레퍼런스

이 레퍼런스는 내장(임베디드) 워크플로 스텝과 설치된 사용자 정의 스텝 타입 모두에
대해 지원되는 Python 계약을 설명합니다. 둘은 같은 `StepBase` 메서드를 구현하고
같은 `StepContext`, `StepResult`, `StepStatus` 타입을 사용합니다. 워크플로 정의와
엔진이 소유하는 실행 의미론은 [워크플로 API 레퍼런스](../workflows/api.md)에
있습니다.

## 전달 경로

내장 스텝은 Specify CLI와 함께 제공되며 별도로 설치되는 `step.yml` 패키지가
필요하지 않습니다. 소스 등록은 내부 기여자의 관심사이지 스텝 구현 API가 아닙니다.

설치된 사용자 정의 스텝은 같은 실행 계약에 더해,
[스텝 개발하기](developing.md)에 문서화된 패키지 메타데이터, 일치하는 타입 정체성,
설치 요구 사항을 사용합니다. 어느 전달 경로도 스텝이 엔진의 레지스트리나
체크포인트 상태를 변경하도록 허가하지 않습니다.

## 스텝 Python API

`StepBase`, `StepContext`, `StepResult`, `StepStatus`를 `specify_cli.workflows.base`에서 임포트하세요.

| API | 계약 |
| --- | --- |
| `StepBase.type_key` | YAML `type`과 일치. 설치된 사용자 정의 스텝은 패키지 정체성과도 일치해야 함 |
| `validate(config) -> list[str]` | 검증 오류. 비어 있으면 유효함 |
| `execute(config, context) -> StepResult` | 상태 없고 스레드에 안전한 실행 |
| `can_resume(state) -> bool` | true를 반환하는 기반 메서드. 현재 엔진은 이를 재개 게이트로 호출하지 않음 |
| `StepContext` | 입력, 이전 결과, 기본값, 경로, 실행 ID, 팬아웃 항목, 재개 컨텍스트 |
| `StepResult.status` | `pending`, `running`, `completed`, `failed`, `skipped`, `paused`. 기본값 `completed` |
| `StepResult.output` | 출력 매핑. 기본값은 빈 매핑 |
| `StepResult.next_steps` | 중첩 확장. 기본값은 빈 목록 |
| `StepResult.error` | 실패 시의 오류 메시지. 그렇지 않으면 null |

`config`는 스텝의 YAML 매핑입니다. 기반 `validate()`는 `id`가 있는지 확인합니다.
오버라이드는 `super().validate(config)`로 그 검사를 보존하고 자신의 오류를
추가해야 합니다. 실행은 검증기를 자동으로 호출하지 않으므로 `execute()`에서도
유효하지 않은 설정을 방어하세요.

### StepContext 필드

| 필드 | 타입 | 의미 |
| --- | --- | --- |
| `inputs` | `dict[str, Any]` | 해석된 워크플로 입력 |
| `steps` | `dict[str, dict[str, Any]]` | 작성된 스텝 ID를 키로 하는 이전 스텝 결과 기록. 출력은 각 기록의 `output` 아래에 있음 |
| `item` | `Any` | 현재 팬아웃 항목. 그렇지 않으면 `None` |
| `inside_fan_out` | `bool` | 실행이 팬아웃 템플릿 안에 있는지 여부 |
| `fan_in` | `dict[str, Any]` | 집계된 팬인 컨텍스트 |
| `default_integration` | `str \| None` | 워크플로 통합 기본값 |
| `default_model` | `str \| None` | 워크플로 모델 기본값 |
| `default_options` | `dict[str, Any]` | 워크플로 디스패치 기본값 |
| `project_root` | `str \| None` | 프로젝트 디렉터리 |
| `run_id` | `str \| None` | 현재 실행 ID |
| `is_resume` | `bool` | 이 실행이 재개 시도인지 여부 |
| `workflow_dir` | `str \| None` | 현재 워크플로 정의의 소스 디렉터리 |
| `default_reasoning_effort` | `str \| None` | 작성된 그대로의 워크플로 `reasoning_effort` 기본값. 평가되지 않은 표현식일 수 있음 |
| `default_context_size` | `str \| None` | 작성된 그대로의 워크플로 `context_size` 기본값. 평가되지 않은 표현식일 수 있음 |

매핑은 빈 딕셔너리, 불리언 플래그는 false, 선택적 경로/식별자는 `None`이
기본값입니다. 경로는 `Path` 객체가 아니라 문자열입니다. 워크플로 기준 상대
리소스에는 `workflow_dir`을, 프로젝트 로컬 작업에는 `project_root`를 사용하세요.
둘이 같은 디렉터리일 필요는 없습니다.

입력과 이전 결과 기록은 컨텍스트로 취급하고, 실행 상태를 다시 쓰는 수단으로
취급하지 마세요. 출력은 `StepResult`를 통해 반환하세요. 팬아웃 항목은 독립적인
결과 컨텍스트를 가지며, 워크플로 호출은 자체 입력과 기본값을 갖습니다.

### 결과와 실패

임의의 상태 문자열 대신 `StepStatus` 값을 사용하세요. 정상적으로 성공한 스텝은
`COMPLETED`를 반환합니다. `SKIPPED`는 건너뛴 스텝을 기록하고, `PAUSED`는 일시
정지를 요청하고, `FAILED`는 오류를 기록합니다. `PENDING`과 `RUNNING`도 열거형에
있지만, 어느 쪽을 반환해도 해당 실행 건이 보류 중이거나 실행 중으로 남지
않습니다. 엔진은 이를 완료로 취급합니다. 진행 상태가 아니라 이후 재개를 위해
멈추려면 `PAUSED`를 반환하세요.

엔진이 출력을 영속화하고 이후의 `{{ steps.<id>.output.* }}` 표현식에 노출하므로
직렬화 가능한 출력 값을 반환하세요. `next_steps`는 제어 흐름 확장을 위한 중첩 스텝
정의 매핑의 목록이지, 스텝 인스턴스의 목록이 아닙니다.

예상된 실패의 경우 관련 진단 정보를 `output`에 담아
`StepResult(status=StepStatus.FAILED, error=...)`를 반환하세요. 그러면 엔진이
`continue_on_error`를 적용할 수 있습니다. 처리되지 않은 예외는 그 플래그와
무관하게 실행을 멈춥니다. 유효하지 않은 설정이나 실패한 작업을 성공한 기본값으로
바꾸지 마세요.

```python
from specify_cli.workflows.base import StepBase, StepContext, StepResult, StepStatus


class GreetingStep(StepBase):
    type_key = "greeting"

    def validate(self, config: dict) -> list[str]:
        errors = super().validate(config)
        if not isinstance(config.get("name"), str) or not config["name"].strip():
            errors.append("'name' must be a non-empty string.")
        return errors

    def execute(self, config: dict, context: StepContext) -> StepResult:
        errors = self.validate(config)
        if errors:
            return StepResult(
                status=StepStatus.FAILED,
                error="; ".join(errors),
            )
        return StepResult(output={"message": f"Hello, {config['name']}!"})
```

## 실행 및 재개 계약

스텝 인스턴스는 동시에 호출될 수 있습니다. 실행을 상태 없이, 스레드에 안전하게
유지하세요. 실행별 값은 `self`의 변경 가능한 속성이 아니라 `config`와
`context`로부터 도출하세요.

엔진이 체크포인트와 라이프사이클 전환을 소유합니다. 완료된 실행 건은 재개 시 구현을
다시 호출하지 않고 재생됩니다. 선택된 분기와 중첩 확장은 고정된 채로 남고, 끝나지
않은 실행 건은 [워크플로 재개 규칙](../workflows/api.md#execution-identity-and-resume)에
따라 계속됩니다.
`can_resume()`을 권한 부여나 안전 검사로 사용하지 마세요. 엔진이 강제하는 게이트가
아닙니다.

완료 체크포인트 이전에 수행된 부수 효과는 중단 후에 반복될 수 있습니다. 외부
작업을 멱등하게, 또는 다른 방식으로 안전하게 재시도할 수 있게 만드세요. 실행은
정확히 한 번(exactly-once)을 보장하지 않습니다. 워크플로 스냅샷은 정의를 동결할
뿐, 스텝 패키지 코드나 외부 리소스를 동결하지 않습니다.

## 설치된 타입 탐색

설치된 각 스텝은 소스의 *종류*만 기록합니다. `catalog`(선택적으로 카탈로그 이름
포함), `local`, `url`입니다. 로컬 경로와 소스 URL은 절대 영속화되지 않습니다.
설치된 타입과 출처를 확인하려면 `specify workflow step list`와
`specify workflow step info <id>`를 사용하세요. 호스트의 내부 Python 레지스트리를
변경하지 마세요.

## 상태 및 설정 위치

| 컴포넌트 | 위치 | 형식 | 용도 |
| --- | --- | --- | --- |
| 사용자 정의 스텝 | `.specify/workflows/steps/{id}/` | 패키지 | 설치된 스텝 매니페스트와 Python |
| 스텝 레지스트리 | `.specify/workflows/steps/step-registry.json` | JSON | 설치된 스텝 메타데이터 |
| 스텝 카탈로그 캐시 | `.specify/workflows/steps/.cache/*.json` | JSON | 캐시된 스텝 카탈로그 항목 |
| 프로젝트 스텝 카탈로그 | `.specify/step-catalogs.yml` | YAML | 프로젝트 수준 스텝 소스 |
| 사용자 스텝 카탈로그 | `~/.specify/step-catalogs.yml` | YAML | 사용자 수준 스텝 소스 |

## 함께 보기

- [사용자 정의 스텝 개발하기](developing.md)
- [내장 스텝 설정](types.md)
- [스텝 CLI](cli.md)
- [스텝 카탈로그](catalogs.md)
- [워크플로 YAML과 실행 의미론](../workflows/api.md)
