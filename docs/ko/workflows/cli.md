# 워크플로 CLI 레퍼런스

`specify workflow` 그룹은 워크플로 패키지와 실행 상태를 관리합니다. 대부분의 명령에는 초기화된 Spec Kit 프로젝트가 필요하며, 로컬 YAML 실행은 예외입니다. 옵션은 명령마다 다릅니다. `--json`은 그룹 전체의 플래그가 아닙니다.

## 워크플로 실행

```bash
specify workflow run <source>
```

| 옵션                | 설명                                                     |
| ------------------- | -------------------------------------------------------- |
| `-i` / `--input`    | 입력 값을 `key=value`로 전달(반복 가능)                  |
| `--json`            | 실행 결과를 단일 JSON 객체로 출력                        |

설치되고 활성화된 워크플로 ID 또는 로컬 YAML 파일을 실행합니다. 카탈로그 워크플로나
URL을 ID로 실행하려면 먼저 `specify workflow add`로 설치하세요. 워크플로가 선언한
입력은 `--input`으로 제공하거나 대화형으로 입력받을 수 있습니다.

예시:

```bash
specify workflow run speckit -i spec="Build a kanban board with drag-and-drop task management"
```

입력 값은 [입력 타입 계약](api.md#input-types)을 따릅니다.

`--json`을 쓰면 형식화된 텍스트 대신 기계 판독 가능한 단일 객체가 출력됩니다(플래그를 생략하면 기본 출력은 바뀌지 않습니다).

```bash
specify workflow run my-pipeline.yml --json
```

```json
{
  "run_id": "662bf791",
  "workflow_id": "build-and-review",
  "status": "paused",
  "current_step_id": "review",
  "current_step_index": 0
}
```

`workflow_id`는 파일 이름이 아니라 YAML 안에 선언된 `workflow.id`입니다. 객체는 표시된 그대로, 즉 두 칸 들여쓰기로 보기 좋게 출력되며 Rich 마크업 없는 일반 stdout으로 출력되므로 항상 파싱됩니다. 워크플로가 `--json`으로 실행되는 동안 스텝이 출력할 진행 상황(예: 게이트 프롬프트, 프롬프트 스텝의 CLI 하위 프로세스 출력)은 stderr로 리디렉션되므로 stdout에는 JSON 객체만 실립니다. 객체는 stdout에서 읽고, stderr은 터미널에 연결된 채로 두거나 따로 캡처하세요.

`failed`와 `aborted` 실행의 경우 페이로드에 종결 스텝의 오류 메시지를 담은 `error` 필드가 포함됩니다.

```json
{
  "run_id": "662bf791",
  "workflow_id": "build-and-review",
  "status": "failed",
  "current_step_id": "boom",
  "current_step_index": 0,
  "error": "Command exited with code 3"
}
```

`completed`와 `paused` 실행은 `error` 필드를 생략합니다. 오류는 실행의 `state.json`에 영속화되므로, 이후에 `specify workflow status <run_id> --json`으로 같은 메시지를 확인할 수 있습니다.

실행 상태를 만들거나 불러오기 전의 어댑터 로드 실패는 실행 전 실패 봉투를
사용합니다. 새 실행에는 실행 ID가 없고, 재개는 제공된 실행 ID를 유지하지만
워크플로나 스텝 컨텍스트가 없습니다. 실행 상태가 존재한 뒤에는, 스텝별 재로드나
검증된 지연 임포트에서 발생한 어댑터 실패가 실행 I/O 실패와 마찬가지로 실제 실행,
워크플로, 현재 스텝 필드를 유지합니다. 상태 저장이 실패하면 디스크의 상태는 보고된
I/O 실패가 아니라 마지막으로 성공한 저장을 여전히 반영할 수 있습니다.

> **참고:** 대부분의 워크플로 명령에는 `specify init`으로 이미 초기화된 프로젝트가 필요합니다. 예외는 프로젝트 밖에서도 실행할 수 있는 `specify workflow run <local-file.{yml,yaml}>`입니다. 그 경우 실행 상태는 현재 디렉터리의 `.specify/workflows/runs/<run_id>/` 아래에 저장됩니다.
>
## 워크플로 재개

```bash
specify workflow resume <run_id>
```

| 옵션                | 설명                                                     |
| ------------------- | -------------------------------------------------------- |
| `-i` / `--input`    | 갱신된 입력 값을 `key=value`로 전달(반복 가능)           |
| `--json`            | 재개 결과를 단일 JSON 객체로 출력                        |

일시 정지되었거나 실패한 워크플로 실행을 영속화된 실행 상태에서 재개합니다.
`running` 상태의 실행은 재개할 수 없습니다. 재개는 스텝 구현을 다시 실행하지 않고
완료된 작업을 표현식 컨텍스트로 재생한 다음, 끝나지 않은 실행 건에서 계속합니다.
덕분에 일시 정지 이후에야 얻게 된 정보나 실패 이후 수정한 값으로 실행을 이어 갈 수
있습니다.

제공된 `--input` 값은 실행에 저장된 루트 입력 위에 병합되고 워크플로의 입력 타입에
대해 다시 검증됩니다. 알 수 없는 루트 입력 이름은 무시됩니다. 갱신된 값은 끝나지
않은 작업에만 영향을 줍니다. 완료된 스텝과 호출은 재해석되거나 반복되지 않습니다.

```bash
specify workflow resume <run_id> --input cmd="exit 0"
```

<a id="workflow-status"></a>

## 워크플로 상태

```bash
specify workflow status [<run_id>]
```

| 옵션                | 설명                                                     |
| ------------------- | -------------------------------------------------------- |
| `--json`            | 실행 상태(또는 실행 목록)를 JSON 객체로 출력             |

특정 실행의 상태를 보여 주거나, ID가 없으면 모든 실행을 나열합니다. 실행 상태: `created`, `running`, `completed`, `paused`, `failed`, `aborted`.

## 워크플로 정의

```bash
specify workflow definition <run_id> --json
```

| 옵션     | 설명                                                             |
| -------- | ---------------------------------------------------------------- |
| `--json` | 필수. 실행의 워크플로 정의를 JSON 객체로 출력                    |

실행이 **시작될 때 사용한** 워크플로 정의를 반환하므로, 통합은 엔진이 실제로 실행하는 것과 어긋나지 않게 실행(예: 선언된 게이트와 그 메시지)을 렌더링할 수 있습니다. 실행 자신의 영속 데이터만 읽고 설치된 워크플로는 절대 읽지 않습니다. 설치된 워크플로를 편집하거나 제거해도 출력은 바뀌지 않습니다. 런타임 진행 상황과 활성 게이트는 [`specify workflow status --json`](#workflow-status)에서 얻습니다.

`--json`을 생략하면 코드 `2`로 종료하고 stderr에 사용법 메시지를 출력하며, stdout에는 아무것도 출력하지 않습니다. 텍스트 렌더러는 미뤄 두었으므로 아래의 JSON 형태가 유일한 계약입니다.

```json
{
  "run_id": "662bf791",
  "definition": {
    "schema_version": "1.0",
    "workflow": { "id": "spec-review", "name": "Spec review", "version": "1.0.0" },
    "steps": [
      { "id": "draft", "type": "command", "command": "speckit.specify" },
      { "id": "approve", "type": "gate", "message": "Review the specification." }
    ]
  },
  "workflow_scopes": []
}
```

- **`definition`**은 실행을 위해 영속화된 워크플로 그대로이며 오버레이가 이미 적용되어 있으므로, 실행된 그대로의 스텝을 반영합니다. 모든 키가 그대로 전달됩니다(`requires`, `outputs`, 알 수 없는 키 포함). 아무것도 추가되거나, 이름이 바뀌거나, 채워지지 않습니다. 게이트의 `options`처럼 생략된 필드는 런타임에 엔진 기본값이 적용된다는 뜻입니다. 표현식(`{{ ... }}`)은 평가되지 않습니다. 구조는 [워크플로 YAML 스키마](api.md#workflow-yaml-schema)를 따르므로 `schema_version`이 이 인터페이스의 버전이며, 스키마 변경은 곧 인터페이스 변경입니다. `definition`은 작성된 그대로 출력되므로 생략된 `schema_version`은 `"1.0"`이 기본값이고 엔진은 `str(schema_version)`을 비교합니다(따라서 YAML의 따옴표 없는 `schema_version: 1.0`은 JSON 숫자 `1.0`으로 나타납니다). 향후 버전에서 최상위 응답 봉투(`run_id`, `definition`, `workflow_scopes`)에 키가 추가될 수 있지만, 기존 봉투 키는 절대 제거되거나 이름이 바뀌지 않습니다.
- **일반 JSON.** 따옴표 없는 날짜처럼 JSON이 표현할 수 없는 YAML 고유 값은 문자열로 나타납니다. 바이너리 값(`!!binary`)은 출력되지 않습니다. 각각 `"<binary: N bytes>"` 자리 표시자로 나타나며 값 자체는 실행의 스냅샷에 남습니다. 집합(`!!set`)은 JSON 형태로 정렬된 목록으로 나타나므로 출력이 해시 무작위화에 좌우되지 않습니다. 문자열이 아닌 매핑 키와 유한하지 않은 숫자도 문자열화됩니다. 같은 JSON 속성 이름으로 합쳐지는 두 매핑 키(예: `1`과 `"1"`)는 손실이 있는 JSON을 내보내는 대신 명령을 실패시킵니다. JSON은 참조를 공유할 수 없으므로 재사용된 YAML 별칭(앵커)은 복제됩니다. 재귀 별칭, 또는 고정된 노드나 출력 크기 예산을 넘어 확장되는 비재귀 별칭 그래프는 CPU, 메모리, stdout을 한없이 소비하는 대신 실패합니다. 예산은 각 정의(루트와 `workflow_scopes`의 모든 항목)에 따로 적용되므로, 바인딩된 워크플로 호출이 많은 실행이 전체 크기 때문에 거부되지는 않습니다. 별도의 훨씬 큰 한도(32 MiB)가 전체 응답을 제한하므로, 큰 정의를 반복하는 루프나 팬아웃은 그것을 증식시키는 대신 실패합니다. 문자열화했을 때 256자를 넘는 문자열이 아닌 매핑 키도 같은 이유로 거부됩니다.
- **`workflow_scopes`**(항상 존재)는 도달한 모든 [워크플로 호출](../workflow-steps/types.md#workflow-composition)의 동결된 정의를 실행 순서대로 담습니다. 각 항목은 `{ "scope_path", "workflow_id", "definition" }`이며 `scope_path`와 `workflow_id`는 `status --json`의 값과 동일합니다. `definition` 안의 호출 스텝은 작성된 그대로 남습니다. 호출 대상이 제자리에서 확장되는 일은 없습니다. 아직 도달하지 않았거나 대상을 해석할 수 없었던 호출은 항목이 없으며, 그 정의는 설치된 워크플로에서 절대 읽지 않습니다. 목록은 호출에 도달할 때마다 늘어나고, 루프나 팬아웃 안의 호출은 반복이나 항목마다 항목 하나를 갖습니다.
- **포함되지 않는 것: 동적으로 생성된 스텝.** 출력에는 작성된 루트 정의와 바인딩된 워크플로 호출의 동결된 정의가 들어 있습니다. 사용자 정의 스텝 타입이 런타임에 `StepResult.next_steps`를 통해 생성하는 스텝은 실행의 실행 상태에 영속화되지만 내보내지지 않으므로, 그렇게 만들어진 게이트는 이 출력에 정의가 없습니다. 내장 `if`, `switch`, `while`, `do-while`, `fan-out` 스텝은 작성된 정의에서 스텝을 가져오므로 포함됩니다.
- **`scope_path`**는 루트부터의 문자열 목록입니다. 감싸는 모든 스텝의 작성된 `id`입니다(`id`가 없는 스텝은 `step-<index>`). `if`, `switch`, 루프, 팬아웃 같은 컨테이너도 요소입니다. 각 루프 반복과 팬아웃 항목은 첫 번째를 포함해 그 인덱스를 문자열로 추가합니다(`["loop", "0", "call"]`). 여기서는 `workflow_scopes`가 항상 존재하지만, `status --json`은 실행에 워크플로 호출이 있을 때만 이를 포함합니다.

### 게이트를 그 정의에 대응시키기

`status --json`은 활성 게이트를 `gate.step_id`와 `gate.scope_path`로 보고합니다. 게이트의 정의를 찾으려면 다음과 같이 합니다.

1. `gate.scope_path`가 없으면 `[]`로 읽습니다. `status --json`은 루트 워크플로에 직접 있는 게이트에 대해 이를 생략합니다. 그런 다음 `scope_path`가 `gate.scope_path`의 접두사인 것 중 가장 긴 `workflow_scopes` 항목을 고릅니다. 없으면 루트 `definition`을 사용합니다. 정확한 일치를 요구하지 마세요. `gate.scope_path`는 `if`, `switch`, 루프, 팬아웃 컨테이너를 포함해 감싸는 모든 스텝을 나열하는 반면, 항목의 `scope_path`는 호출 스텝에서 끝납니다. `route`라는 `if` 안에 있는 `call`이 호출한 워크플로 안의, `wrap`이라는 `if`에 있는 게이트는 `["route", "call", "wrap"]`을 보고하고, 그 항목은 `["route", "call"]`을 갖습니다.
2. 그 정의에서 `id`가 작성된 스텝 ID와 같은 스텝을 찾습니다. 이후의 루프 반복과 팬아웃 항목에서 `step_id`는 `<container>:<id>:<n>`으로 한정됩니다(예: `fan:g:0`). 작성된 스텝 ID에는 `:`가 허용되지 않으므로 작성된 ID는 `:`로 구분된 세그먼트 중 끝에서 두 번째입니다. 스텝 ID는 자체 ID 네임스페이스를 가진 팬아웃 템플릿 사이를 제외하면 정의 안에서 고유합니다. 루트 스텝과 팬아웃 템플릿 스텝은 ID를 공유할 수 있습니다. `gate.scope_path`가 팬아웃 스텝을 지나간다면 그 스텝의 `step` 템플릿 안만 살펴보고, 그렇지 않으면 팬아웃 템플릿을 무시하세요. 예를 들어 루트 셸 `g`와 팬아웃 `fan`의 템플릿에 있는 게이트 `g`가 모두 존재할 때, `scope_path`가 `["fan", "0"]`인 `fan:g:0`은 루트 셸이 아니라 템플릿의 게이트입니다.

사용자 정의 스텝이 런타임에 생성한 게이트에는 대응시킬 정의가 없습니다. 위의 동적으로 생성된 스텝에 대한 설명을 참고하세요.

`status --json`은 호출된 워크플로 안의 스텝에 대해 스텝별 상태를 보고하지 않습니다.

### 오류

실패는 단일 `{"error": "<message>"}` 객체를 stderr에 쓰고, stdout을 비워 두고, 코드 `1`로 종료합니다. Spec Kit 프로젝트가 아님, 유효하지 않은 `SPECIFY_INIT_DIR`, 안전하지 않은(심볼릭 링크 포함) 워크플로 스토리지, 유효하지 않거나 알 수 없는 실행 ID, 읽을 수 없거나 너무 깊게 중첩된 실행 상태, 영속화된 정의가 없는 실행(예: 정의가 영속화되기 전에 만들어진 것), 손상된 정의나 실행 상태, 매핑 키가 같은 JSON 속성 이름으로 합쳐지는 정의, 재귀적이거나 확장하기에 너무 큰 YAML 별칭을 가진 정의가 해당합니다. `--json`을 쓴 사용법 오류도 이 JSON 오류 계약을 사용하며 코드 `2`로 종료합니다.

## 설치된 워크플로 나열

```bash
specify workflow list
```

현재 프로젝트에 설치된 워크플로를 나열합니다.

<a id="install-a-workflow"></a>

## 워크플로 설치

```bash
specify workflow add <source>
```

| 옵션            | 설명                                                   |
| --------------- | ------------------------------------------------------ |
| `--dev`         | 로컬 YAML 파일, 패키지 디렉터리, 아카이브에서 설치 |
| `--from <url>`  | 사용자 정의 URL에서 설치(`<source>`는 예상되는 워크플로 ID를 지정) |
| `--version <version>` | 공개된 정확한 카탈로그 릴리스를 설치(`<source>`는 워크플로 ID여야 함) |

카탈로그, HTTPS URL, 로컬 YAML 파일, `workflow.yml`을 담은 디렉터리, 또는 `.zip`,
`.tar.gz`, `.tgz` 아카이브로부터 워크플로를 설치합니다. 아카이브는 `workflow.yml`을
루트 또는 하나의 최상위 디렉터리 안에 담을 수 있습니다.

디렉터리와 아카이브 설치는 스크립트와 그 밖의 동반 파일을 포함해 워크플로 패키지
전체를 보존합니다. ZIP, `.tar.gz`, `.tgz` 아카이브는 같은 검증 및 설치 동작을
따릅니다.

정확한 버전의 카탈로그 선택과 그 다이제스트 요구 사항은 [카탈로그 릴리스](catalogs.md#workflow-releases)를 참고하세요.

## 워크플로 함수

```bash
specify workflow function add <id> --dev <directory>
specify workflow function add <id> --from <archive-url>
specify workflow function list [--json]
specify workflow function remove <id>
```

`add`에는 소스 옵션이 정확히 하나 필요합니다. 함수 패키지는 Python 코드이며
워크플로 검증 전에 로드됩니다. 검토한 패키지만 설치하세요. `list --json`은 패키지
코드를 임포트하지 않고 레지스트리 메타데이터와 범용 export 사용법을 보고합니다.

## 워크플로 업데이트

```bash
specify workflow update [workflow_id]
```

설치된 카탈로그 워크플로 하나를, 또는 ID가 없으면 전부를 최신 카탈로그 버전으로 업데이트합니다. 확인을 요청하며, 다운로드나 검증이 실패하면 설치된 사본을 유지합니다.

## 워크플로 활성화 또는 비활성화

```bash
specify workflow enable <workflow_id>
specify workflow disable <workflow_id>
```

비활성화된 워크플로는 설치되고 목록에 나타난 채로 남지만(`[disabled]`로 표시) 다시 활성화될 때까지 실행을 거부합니다.

## 워크플로 제거

```bash
specify workflow remove <workflow_id>
```

설치된 워크플로를 프로젝트에서 제거합니다.

## 사용 가능한 워크플로 검색

```bash
specify workflow search [query]
```

| 옵션       | 설명              |
| ---------- | ----------------- |
| `--tag`    | 태그로 필터링     |
| `--author` | 작성자로 필터링   |

모든 활성 카탈로그에서 쿼리와 일치하는 워크플로를 검색합니다.

## 워크플로 정보

```bash
specify workflow info <workflow_id>
specify workflow info <workflow_id> --versions
```

스텝, 입력, 요구 사항을 포함해 워크플로에 대한 상세 정보를 보여 줍니다.
`--versions`는 현재 카탈로그 버전에 이어 공개된 과거 버전을 나열하고, 이기는
카탈로그가 설치 가능한지 탐색 전용(설치 불가)인지 알려 줍니다. 로컬에 다른 버전이
설치되어 있어도 동작합니다.

## 워크플로 해석

```bash
specify workflow resolve <workflow-id>
```

기반/오버레이 스택과 스텝 소스 귀속을 보여 줍니다. 전체 셸 명령이나 환경 바인딩은 표시하지 않습니다. 실행 전에 기여하는 소스 파일을 검토하세요.

<a id="overlay-management"></a>

## 오버레이 관리

### 프로젝트 오버레이 추가

```bash
specify workflow overlay add <path-to-overlay.yml> --priority <n>
```

오버레이 파일을 검증하고 `.specify/workflows/overlays/<extends>/<id>.yml`로 복사합니다. `--priority`는 기본값이 `10`이며 파일의 `priority` 필드를 재정의합니다.

### 오버레이 나열

```bash
specify workflow overlay list <workflow-id>
```

워크플로의 모든 오버레이를 해석기 우선순위 순으로 보여 줍니다. 비활성화된 오버레이는 목록에서 비활성으로 표시되며 워크플로 해석 중에 무시됩니다.

### 우선순위 변경

```bash
specify workflow overlay set-priority <workflow-id> <overlay-id> <n>
```

### 활성화 또는 비활성화

```bash
specify workflow overlay disable <workflow-id> <overlay-id>
specify workflow overlay enable <workflow-id> <overlay-id>
```

### 제거

```bash
specify workflow overlay remove <workflow-id> <overlay-id>
```

프로젝트 오버레이 파일을 제거합니다.

<a id="catalog-management"></a>

## 카탈로그 관리

### 카탈로그 나열

```bash
specify workflow catalog list
```

모든 활성 카탈로그 소스를 보여 줍니다.

### 카탈로그 추가

```bash
specify workflow catalog add <url> [--name <name>]
```

| 옵션            | 설명                             |
| --------------- | -------------------------------- |
| `--name <name>` | 카탈로그의 선택적 이름           |

사용자 정의 카탈로그 URL을 프로젝트의 `.specify/workflow-catalogs.yml`에
추가합니다. 새 항목은 설치를 허용하며 기존 항목 다음의 우선순위를 받습니다. 검증한
소스만 추가하세요. 탐색 전용 소스는 카탈로그 설정을 편집해 구성하세요.

같은 워크플로 카탈로그 URL을 같은 이름으로 다시 추가하면 설정 변경 없이 성공합니다. 다른 이름은 거부됩니다.

### 카탈로그 제거

```bash
specify workflow catalog remove <index>
```

해당하는 우선순위 순 `catalog list`의 인덱스로 프로젝트 카탈로그를 제거합니다.
`SPECKIT_WORKFLOW_CATALOG_URL`이 제공하는 소스는 이 방법으로 제거할 수 없습니다. 프로젝트 소스를 관리하려면 그 변수의 설정을 해제하세요.

## 함께 보기

- [워크플로 개요](overview.md)
- [워크플로 API](api.md)
- [스텝 관리 CLI](../workflow-steps/cli.md)
- [내장 스텝 타입](../workflow-steps/types.md)
- [오버레이 형식](overlays.md)
- [카탈로그 스키마와 신뢰](catalogs.md)
