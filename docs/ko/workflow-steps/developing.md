# 워크플로 스텝 개발하기

사용자 정의 스텝 타입은 `specify workflow step`으로 설치합니다. 스텝은
메타데이터와 실행 가능한 Python을 담은 디렉터리 패키지입니다.

<a id="custom-step-packages"></a>

## 사용자 정의 스텝 패키지

```text
my-step/
├── step.yml        # required, at the package root
├── __init__.py     # required, at the package root
└── helpers.py      # optional nested modules and data files
```

`step.yml`은 스텝의 정체성을 선언합니다. `step.type_key`는 명령줄에 전달하는
`<step_id>`와 정확히 일치해야 합니다. ID는 패키지 내용으로부터 추론되지 않습니다.

```yaml
step:
  type_key: my-step
  name: My Step
  version: 0.1.0
  author: you
  description: What this step does
```

`__init__.py`는 `type_key`가 일치하는 `StepBase` 하위 클래스를 정의해야 합니다.

```python
from specify_cli.workflows.base import StepBase, StepResult


class MyStep(StepBase):
    type_key = "my-step"

    def execute(self, config, context):
        return StepResult(output={"ok": True})
```

### 패키지 검증

모든 소스는 무언가가 커밋되기 전에 동일하게 검증됩니다.

- `step.yml`과 `__init__.py`는 패키지 루트에 있는, 심볼릭 링크가 아닌 일반
  파일이어야 합니다.
- 패키지 트리는 재귀적으로 복사됩니다(상대 임포트, 중첩된 헬퍼 모듈, 데이터 파일이
  지원됨). 심볼릭 링크된 패키지 루트, 하위의 모든 심볼릭 링크, 일반 파일이나
  디렉터리가 아닌 모든 파일시스템 객체는 거부됩니다.
- `.git`, `__pycache__`, `.DS_Store` 항목은 검사 없이 건너뜁니다. 복사되지 않고,
  제외된 디렉터리로는 들어가지 않으며, 어떤 한도에도 포함되지 않습니다.
- 설치 패키지 정책은 최대 **512개의 유지 항목**(파일과 디렉터리 합산), 최대
  **32단계**의 디렉터리 중첩, **50 MiB**의 유지 콘텐츠를 허용합니다.
- 아카이브 URL은 패키지 검증 전에 전송/추출 안전 한도도 통과해야 합니다. 아카이브
  항목 최대 512개, 다운로드 또는 추출 50 MiB, 아카이브 멤버당 10 MiB입니다.
  카탈로그 파일에는 응답당 50 MiB 제한이 있습니다.
- 설치는 패키지를 검증하고 복사하지만 `__init__.py`를 임포트하거나 실행하지
  **않습니다**. 설치된 사용자 정의 스텝 모듈은 `workflow add`, `workflow run`,
  `workflow resume`의 시작 과정에서, 특정 사용자 정의 스텝이 반드시 실행되기 전에
  로드됩니다.

> **보안 참고:** 사용자 정의 스텝을 로드하면 그 Python이 **여러분의** 권한으로
> 실행됩니다. 신뢰하는 소스의 스텝 패키지만 설치하고 유지하세요.

## 사용자 정의 스텝 구현하기

[Python API 레퍼런스](api.md)를 따르세요. `StepBase`를 상속하고, 패키지와 구현에
같은 `type_key`를 부여하고, 명시적인 상태, 출력, 오류를 반환합니다. `validate()`를
오버라이드하고 `execute()`에서도 유효하지 않은 설정을 방어하세요. 실행은 검증기를
자동으로 호출하지 않습니다.

스텝 인스턴스는 동시 팬아웃 중에 여러 스레드에서 호출될 수 있으므로 실행을 상태
없이, 스레드에 안전하게 유지하세요. 실행별 데이터는 `self`가 아니라 `config`와
`context`에 저장하세요. 체크포인트와 라이프사이클 전환은 엔진이 소유하게 두세요.

로컬 반복 작업에는 `specify workflow step add my-step --dev ./my-step`을 사용하고, 편집 후에는 `--force`를 사용하세요. 소스 검증과 교체 실패 동작은 [스텝 설치 레퍼런스](cli.md#step-management)를 참고하세요.

## 로컬에서 테스트하기

유효한 설정과 유효하지 않은 설정, 반환되는 상태, 출력, 오류를 테스트하세요.
관련 재개, 중첩 스텝, 동시 실행 경로를 포함해 작은 워크플로를 통해 타입을 실행해
보세요. 패키지나 워크플로가 부수 효과를 낼 수 있다면 일회용 초기화 프로젝트를
사용하세요.

## 번들 로컬 한계

번들의 `provides.steps`는 여전히 활성 스텝 카탈로그를 통해서만 해석됩니다.
번들 로컬 `steps/<id>/` 페이로드와 상대 `provides.steps[].source` 재정의는 이
릴리스에서 해석되지 **않으므로**, 그런 스텝은 오프라인으로 설치할 수 없습니다.
[번들 오프라인 한계](../bundles/cli.md#offline-limits)를 참고하세요.

## 함께 보기

- [스텝 개요](overview.md)
- [Python API 레퍼런스](api.md)
- [스텝 설치 CLI](cli.md)
- [스텝 게시하기](publishing.md)
- [커뮤니티 스텝 제출](../community/workflow-steps.md)
- [워크플로 실행과 재개](../workflows/api.md#execution-model)
