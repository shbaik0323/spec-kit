# 워크플로 함수

워크플로 함수는 프로젝트에 설치되어 워크플로 표현식에서 파이프 필터로 노출되는
Python 호출 가능 객체입니다. `.specify/workflows/functions/<id>/` 아래에 설치되며
`function.yml` 매니페스트와 `__init__.py` 모듈로 구성됩니다.

## 함수 설치 및 관리

```bash
specify workflow function add text --dev ./workflow-functions/text
specify workflow function list
specify workflow function list --json
specify workflow function remove text
```

패키지는 아카이브 URL에서 설치할 수도 있습니다.

```bash
specify workflow function add text --from https://example.test/text.zip
```

`--dev`와 `--from` 중 정확히 하나가 필요합니다. `--force`는 같은 패키지 ID를 가진
기존 패키지를 교체합니다. 다른 패키지가 소유한 함수 export는 교체할 수 없습니다.

설치는 사용자 정의 스텝 패키지와 같은 규칙을 따릅니다. 파일은 비공개 스테이징
디렉터리를 거쳐 검증되고 복사되며, 심볼릭 링크와 지원되지 않는 파일 유형은
거부되고, `.git`, `__pycache__`, `.DS_Store`는 복사되지 않으며, 패키지는 최대
512개 항목, 50 MiB, 32단계 디렉터리를 담을 수 있습니다. `--from` 설치는 요청 전에
URL과 함께 신뢰할 수 없는 소스 프롬프트를 표시하고, GitHub 릴리스 자산 URL을
해석하며, URL, 최종 URL, `Content-Type`이 아카이브 형식에서 일치할 것을
요구합니다.

`remove`는 설치와 같은 프로젝트 잠금 아래에서 패키지 디렉터리와 그 레지스트리
항목을 삭제합니다. 레지스트리 항목이 없는 디렉터리는 경고와 함께 제거됩니다.
디렉터리를 삭제할 수 없으면 레지스트리 항목이 복원됩니다.

`list`는 영속화된 레지스트리만 읽습니다. 패키지 코드를 임포트하지 않으며 각
export의 범용 사용 형식을 보고합니다. `.specify/workflows/functions/`에 손으로
복사한 패키지는 워크플로가 실행될 때 로드되지만, 레지스트리 항목이 없으므로 목록에
나타나지 않습니다.

## 매니페스트

```yaml
id: text
name: Text helpers
version: 1.0.0
description: Small text transformations.
functions:
  - name: trim
    handler: trim
    min_args: 0
    max_args: 0
    description: Remove surrounding whitespace.
```

패키지 ID는 사용자 정의 워크플로 스텝과 같은 안전한 경로 구성 요소 검증을
사용합니다. export 이름은 소문자 식별자(`a-z`, `0-9`, `_`)이며 패키지 안에서
고유해야 합니다. `min_args`와 `max_args`는 `0 <= min_args <= max_args <= 16`인
정수이며, 파이프로 전달되는 값 뒤의 인자를 셉니다. 기본값은 0입니다. 괄호 없는
필터는 `min_args`가 0일 때만 허용됩니다. 괄호를 쓴 호출은 인자를 하나 이상 담아야
하며 선언된 범위 안에 있어야 합니다.

핸들러는 `handler(value, *args)`로 호출됩니다. 예:

```python

def trim(value):
    return value.strip()


def surround(value, left, right):
    return f"{left}{value}{right}"
```

```yaml
- id: normalize
  type: shell
  strict: true
  env:
    VALUE: "{{ inputs.value | trim }}"
  run: 'printf "%s\\n" "$VALUE"'
```

## 로딩과 신뢰

설치는 패키지 파일을 검증하고 복사하지만 임포트하지 않습니다. 워크플로를 실행,
재개, 추가, 업데이트하면 워크플로 검증 전에 설치된 함수 모듈이 로드됩니다. 로딩은
Python 코드를 실행하므로 패키지를 검토하고 신뢰하는 소스에서만 설치하세요.
아카이브 다운로드는 다른 실행 가능한 워크플로 패키지와 같은 HTTPS 및 루프백 HTTP
제한을 사용합니다.

함수 이름은 내장 필터 및 설치된 다른 필터와 평면적인 네임스페이스를 공유합니다.
설치는 내장 필터이거나 다른 패키지가 이미 제공하는 이름을 거부합니다. 그 패키지가
레지스트리에 있든 디스크에만 있든 마찬가지입니다(후자의 경우 오류는 "found on
disk, not in the registry"라고 알립니다). 패키지를 수동으로 복사했거나 새 Spec Kit
릴리스가 충돌하는 내장 필터를 도입한 경우, 런타임 로딩도 충돌하는 모든 함수와 두
소스를 알리며 충돌에서 실패합니다. 충돌 검사는 패키지 코드가 임포트되기 전에
이루어집니다. 내장 필터는 절대 대체되지 않습니다. functions 디렉터리 자체를 읽을
수 없으면 명령은 오류를 보고하고 코드 1로 종료합니다(`--json`에서는 실패 페이로드).
매니페스트가 유효하지 않거나 읽을 수 없는 패키지, 또는 모듈 임포트에 실패한
패키지는 건너뛰므로, 그 필터는 나중에 알 수 없는 것으로 보고될 수 있습니다.

필터 레지스트리는 워크플로가 로드되어 있는 동안 프로세스 전역입니다. 프로젝트를
다시 로드하면 이전 프로젝트의 사용자 정의 필터가 대체됩니다. 팬아웃 스텝이
표현식을 동시에 평가할 수 있으므로 함수 핸들러는 상태가 없고 스레드에 안전해야
합니다. 다른 워크플로 부수 효과와 마찬가지로, 재개 체크포인트 이전의 중단은 작업이
다시 실행되게 할 수 있습니다. 재개는 정확히 한 번이 아니라 최소 한 번의 실행을
제공합니다.

사용자 정의 반환 값은 기존의 표현식 및 스텝 출력 처리를 사용합니다. 추가적인 함수
전용 타입이나 JSON 허용 목록은 적용되지 않습니다. 핸들러가 발생시킨 예외는 함수
이름이 앞에 붙은 `ValueError` 메시지로 정규화되며, `BaseException` 하위 클래스는
가로채지 않습니다.
