# 아티팩트

**아티팩트**는 Spec Kit이 프로젝트에 노출하는 모든 명령, 템플릿, 스크립트, 훅을 가리키며, 어느 레이어가 제공하는지는 상관없습니다. 내장 자산, 설치된 프리셋, 설치된 확장, `.specify/templates/overrides/`의 프로젝트 로컬 오버라이드 모두 해당합니다.

`specify artifact` 명령 그룹은 이 목록을 들여다보는 읽기 전용 조회 표면입니다. `specify preset resolve <name>`은 "프리셋이 관리하는 이 이름에 대해 어느 파일이 이기는가?"에 답하고, `specify artifact`는 "애초에 무엇이 존재하며, 그 뒤의 전체 합성 스택은 무엇인가?"에 답합니다. 어떤 프리셋도 건드리지 않는 내장 아티팩트도 포함합니다.

현재 모든 하위 명령은 `--json`을 요구합니다. 생략하면 코드 `2`로 종료하고 stderr에 사용법 메시지를 출력하며, stdout에는 아무것도 출력하지 않습니다. 텍스트 렌더링은 의도적으로 미뤄 두었습니다. 그래야 아래의 JSON 형태가 유일한 계약이 되고, 나중에 기본 텍스트 렌더러를 추가하더라도 호환성을 깨지 않는 추가 변경으로 남기 때문입니다.

## 아티팩트 나열

```bash
specify artifact list --json
```

| 옵션     | 설명                                                     |
| -------- | -------------------------------------------------------- |
| `--json` | 필수. 목록을 JSON 배열로 stdout에 출력합니다.            |

보이는 모든 아티팩트의 전체 목록을 출력합니다. `(kind, name)` 쌍마다 한 행이며, 합성 `stack`을 포함합니다. 이름 있는 아티팩트는 kind(`command`, `template`, `script` 순)와 이름으로 정렬됩니다. 훅 행은 그 뒤에 오며, 이벤트로 정렬한 다음 첫 번째 선언의 우선순위로 정렬됩니다.

```json
[
  {
    "id": "command:speckit.specify",
    "name": "speckit.specify",
    "kind": "command",
    "description": "Create or update the feature specification.",
    "stack": [
      {
        "id": "command:speckit.specify",
        "layer": null,
        "sourceId": null,
        "presetId": null,
        "presetName": null,
        "strategy": "replace",
        "active": true,
        "hidden": false,
        "manifestPath": null,
        "lookupId": null,
        "sourcePath": null
      }
    ]
  },
  {
    "id": "script:setup-plan",
    "name": "setup-plan",
    "kind": "script",
    "description": "Setup implementation plan for a feature.",
    "stack": [
      {
        "id": "script:setup-plan",
        "layer": null,
        "sourceId": null,
        "presetId": null,
        "presetName": null,
        "strategy": "replace",
        "active": true,
        "hidden": false,
        "manifestPath": null,
        "lookupId": null,
        "sourcePath": null
      }
    ]
  }
]
```

| 필드          | 설명                                                                      |
| ------------- | ------------------------------------------------------------------------- |
| `id`          | `{kind}:{name}` — `artifact info`가 인자로 받는 축약 표기                  |
| `name`        | 논리적 아티팩트 이름(명령은 `speckit.<stem>` 네임스페이스를 사용)          |
| `kind`        | `command`, `template`, `script`, `hook` 중 하나                            |
| `description` | 설명을 선언한 최상위 우선순위 레이어의 설명. 없으면 `""`                   |
| `stack`       | 이 아티팩트의 합성 스택. `artifact info`와 같은 행 형태를 사용             |

내장 아티팩트는 아무것도 오버라이드하지 않아도 항상 나타납니다. 설명은 설명을 가진 최상위 우선순위 레이어에서 가져옵니다. 내장 명령을 가리는 프리셋이나 프로젝트 오버라이드는 가려진 내장 텍스트가 아니라 자신의 설명을 보고합니다. 스킬(`.github/skills/**/SKILL.md`)은 제외됩니다. 스킬은 통합별 출력물이지 제공되는 자산군이 아니기 때문입니다.

## 아티팩트 정보

```bash
specify artifact info <name> --json
```

| 옵션             | 설명                                                                |
| ---------------- | ------------------------------------------------------------------- |
| `--json`         | 필수. 합성 스택을 JSON 객체로 stdout에 출력합니다.                  |
| `--kind <kind>`  | 조회를 `command`, `template`, `script`, `hook`으로 좁힙니다          |

`<name>`은 이름만(`speckit.specify`) 또는 `kind:name` 축약 표기(`command:speckit.specify`)를 받습니다. 축약 표기와 `--kind`를 모두 제공하면 둘이 일치해야 합니다.

```json
{
  "id": "command:speckit.specify",
  "name": "speckit.specify",
  "kind": "command",
  "description": "Create or update the feature specification.",
  "stack": [
    {
      "id": "command:speckit.specify",
      "layer": "preset",
      "sourceId": "compliance",
      "presetId": "compliance",
      "presetName": "Compliance Preset",
      "strategy": "replace",
      "active": true,
      "hidden": false,
      "manifestPath": ".specify/presets/compliance/preset.yml",
      "lookupId": "preset:compliance:command:speckit.specify",
      "sourcePath": ".github/skills/speckit-specify/SKILL.md"
    },
    {
      "id": "command:speckit.specify",
      "layer": null,
      "sourceId": null,
      "presetId": null,
      "presetName": null,
      "strategy": "replace",
      "active": false,
      "hidden": true,
      "manifestPath": null,
      "lookupId": null,
      "sourcePath": null
    }
  ]
}
```

최상위 `id`, `name`, `kind`, `description`, `stack` 필드는 `artifact list --json`의 해당 행과 일치합니다.

### 스택 의미론

명령, 템플릿, 스크립트 아티팩트의 `stack`은 해석 우선순위 순으로 정렬됩니다. 인덱스 `0`이 이기는 레이어입니다. 훅 스택은 승자 기반이 아니라 가산적입니다. 여러 항목이 활성 상태일 수 있으며, 선언 순서는 [훅 아티팩트](#hook-artifacts)에서 설명합니다. 각 행은 기여하는 레이어 하나를 설명합니다.

| 필드           | 설명                                                                             |
| -------------- | -------------------------------------------------------------------------------- |
| `id`           | `{kind}:{name}` — 소스와 무관한 왕복(round-trip) 키. 같은 아티팩트 스택의 모든 행에서 동일 |
| `layer`        | `project`, `preset`, `extension`. 내장 레이어는 `null`                            |
| `sourceId`     | 해석기가 사용하는 설치된 프리셋 또는 확장 ID. 레이어에 출처가 없으면 `null`      |
| `presetId`     | 프리셋 팩 디렉터리 id. 내장, `project`, `extension` 행에서는 `null`               |
| `presetName`   | 매니페스트가 선언한 프리셋 표시 이름. 없으면 팩 id. `presetId`가 `null`이면 `null` |
| `strategy`     | `replace`, `wrap`, `prepend`, `append`, `additive`                                |
| `active`       | 레이어가 활성 상태인지 여부. 이름 있는 아티팩트에서는 인덱스 `0`에서만 `true`    |
| `hidden`       | 더 낮은 인덱스의 `replace` 레이어가 이 레이어를 합성에서 잘라낼 때 `true`        |
| `manifestPath` | 선언하는 매니페스트의 프로젝트 상대 경로. 해당 사항이 없으면 `null`              |
| `lookupId`     | 결정적인 `{layer}:{sourceId}:{kind}:{name}` 식별자. 내장 레이어는 `null`         |
| `sourcePath`   | 레이어를 뒷받침하는 실제 파일의 프로젝트 상대 POSIX 경로. 내장/합성 레이어는 `null` |

`active`와 `hidden`은 서로 반대가 아니라 독립적인 레이블입니다. 명령, 템플릿, 스크립트 아티팩트에서 `active`는 기존 Spec Kit 레이어 해석 순서가 선택한 최상위 우선순위 레이어를 식별합니다. 레이어 내용을 읽거나 합성할 수 있는지는 검증하지 않습니다. 이는 이후 내용 합성에서 경고가 발생하더라도 발견된 레이어 체인을 보고하는 `specify preset resolve`의 진단 동작을 보존합니다. 합성 전략(`wrap`, `prepend`, `append`)은 하위 레이어를 합성 출력에 유지하므로, 비활성 레이어가 반드시 숨겨지는 것은 아닙니다. 첫 번째 `replace` 레이어 아래의 레이어만 `hidden`으로 표시됩니다. 내장 행에는 출처가 없습니다. `layer`, `sourceId`, `lookupId`가 `null`입니다. 하지만 `id`는 내장 행에서도 항상 채워집니다. `id`는 왕복 키입니다. `specify artifact info`가 이를 입력으로 받으며(예: `specify artifact info command:speckit.specify --json`), 호출자가 이름만 전달하든 `id`를 전달하든 같은 아티팩트로 해석됩니다.

여기서 **출처(provenance)**란 아티팩트 레이어 하나의 기원을 뜻합니다. 레이어를 제공한
설치된 프리셋이나 확장, 이를 선언한 매니페스트, 이를 뒷받침하는 실제 파일입니다.
lookup ID는 설치된 프리셋 또는 확장 ID를 사용하며, 기존 해석기가 쓰는 정체성 및
순서와 일치합니다. 설치된 ID는 레지스트리/디렉터리 이름이며 매니페스트 안에 선언된
논리적 `id`와 다를 수 있습니다. 설치된 ID를 사용하면 매니페스트가 같은 논리적 ID를
선언하더라도 별도로 설치된 디렉터리가 구분된 채로 유지됩니다.

프로젝트 로컬 오버라이드는 합성된 `project:_:{kind}:{name}` ID를 가지며, 내장
레이어에는 `lookupId`가 없습니다. 이 값들은 아티팩트 스택의 출처를 식별합니다.
아티팩트 왕복 키가 아니므로, 그 용도로는 `artifact info`에 `id`를 사용하세요.
`manifestPath`는 선언하는 매니페스트를 식별하고, `sourcePath`는 레이어를
뒷받침하는 실제 파일이 있을 때 이를 식별합니다. 코어, 프로젝트 오버라이드, 그 밖의
합성 행은 `sourcePath: null`을 보고할 수 있습니다.

## 기여 조회

```bash
specify artifact lookup <lookupId> --json
```

매니페스트가 뒷받침하는 스택 `lookupId`를 Spec Kit이 사용하는 검증된 프리셋 또는
확장 선언으로 해석합니다. 반환되는 선언은 정식 확장 명령 이름, 빈 별칭 목록 같은
주입된 기본값, 소문자 프리셋 전략을 포함해 일반적인 매니페스트 처리를 반영합니다.
작성된 YAML을 그대로 표현한 것이 아닙니다.

이렇게 하면 교차 참조 동작이 artifact 명령 안에 머뭅니다. 프리셋과 확장의 명령,
매니페스트, 해석기 반환 형태는 바뀌지 않습니다.

응답에는 안정적인 lookup ID, 제공자 좌표, 매니페스트 및 소스 경로, 그리고
`contribution` 아래의 유효 선언이 포함됩니다. 관례만으로 이루어진 기여와 프로젝트
오버라이드에는 원본 매니페스트 선언이 없으므로, 조회는 종료 코드 `1`과 함께
`{"error": "unknown contribution <lookupId>"}`를 반환합니다. 내장 행에는
`lookupId`가 없습니다.

<a id="hook-artifacts"></a>

### 훅 아티팩트

훅 행은 표준 프리셋 및 확장 해석기가 포함한 확장의 훅 선언을 투영합니다. 왕복 축약 표기는 다음과 같습니다.

```text
hook:{encodedEventName}:{encodedTargetCommand}
```

예를 들어 다음 둘은 같은 훅 아티팩트를 선택합니다.

```bash
specify artifact info hook:before_specify:speckit.compliance.pre-check --json
specify artifact info before_specify:speckit.compliance.pre-check --kind hook --json
```

훅 이벤트와 대상 명령 구성 요소는 URL 인용 방식으로 UTF-8 퍼센트 인코딩됩니다. ASCII 문자, 숫자, `-`, `.`, `_`, `~`는 문자 그대로 남고, 그 밖의 모든 바이트는 인코딩됩니다. 이 인코딩은 아티팩트 `id`, `name`, `lookupId`의 훅 구성 요소에만 한정됩니다. 훅 매니페스트, 런타임 바인딩, `eventName`, `targetCommand`는 바뀌지 않습니다. 예를 들어 `/skill:speckit-test-ext-hello`를 대상으로 하는 `custom:after`라는 이벤트의 아티팩트 ID는 다음과 같습니다.

```text
hook:custom%3Aafter:%2Fskill%3Aspeckit-test-ext-hello
```

훅 행에는 최상위 필드 세 개가 추가됩니다.

| 필드            | 설명                                                                           |
| --------------- | ------------------------------------------------------------------------------ |
| `eventName`     | 선언을 트리거하는 훅 이벤트                                                    |
| `targetCommand` | 훅 선언이 호출하는 명령                                                        |
| `registered`    | 선언하는 스택 항목 중 하나 이상이 활성화된 런타임 바인딩을 가지면 `true`       |

각 훅 스택 항목에는 선언의 정규화된 `priority`와 `optional` 값도 포함됩니다. 훅 항목은 항상 `strategy: "additive"`와 `hidden: false`를 사용합니다. 여러 확장이 같은 이벤트-명령 쌍을 선언하고 등록할 수 있으며, 활성화된 모든 선언이 활성 상태로 남습니다. 스택 항목은 훅 우선순위로 정렬되고, 해석기의 결정적인 선언 순서가 동률 처리 기준으로 사용됩니다. 이 순서는 선언 목록을 설명하는 것이며, 우선순위가 같은 훅이 런타임에 같은 순서로 실행된다는 약속이 아닙니다.

훅의 경우 `active`는 조건 평가가 아니라 `.specify/extensions.yml`의 등록 상태를 보고합니다. 선언은 런타임 설정에 같은 확장 id, 이벤트, 명령을 가진 활성화된 바인딩이 있을 때만 활성입니다. 조건은 나중에 훅이 실행될 때 평가됩니다. 따라서 런타임 설정이 없거나, 형식이 잘못되었거나, 읽을 수 없으면 선언은 `registered: false`와 `active: false`로 보이는 상태로 남습니다.

선언되었지만 등록되지 않은 훅은 해당 확장이 일반 해석기에 의해 포함되면 계속 보입니다. 레지스트리에서 비활성화된 확장은 다른 기여와 마찬가지로 완전히 제외됩니다. 유효하지 않은 개별 확장 매니페스트도 기존 해석기가 생략하며, 확장 검사 및 검증 명령으로 진단할 수 있습니다.

훅 lookup ID는 아티팩트 전용 `{layer}:{sourceId}:hook:{encodedEventName}:{encodedTargetCommand}` 문법을 사용하며, 여기서 `sourceId`는 설치된 제공자 ID입니다. 훅 출처는 `preset`과 `extension` 레이어로 제한됩니다. 훅은 내장/코어 레이어를 받지 않습니다. 현재 매니페스트 API는 확장 훅 선언을 노출하므로 현재 행은 `extension` 레이어를 사용합니다. `preset` 레이어는 프리셋 제공 훅을 위해 훅 식별자 문법에 예약되어 있으며, 이를 위해 프리셋이나 확장 매니페스트 API에 아티팩트 ID를 추가할 필요는 없습니다.

## JSON 오류

실패 시 stdout에는 아무것도 기록되지 않습니다. 단일 키 JSON 봉투가 stderr에 기록되고 프로세스는 코드 `1`로 종료합니다.

```json
{ "error": "unknown artifact command:nope" }
```

| 메시지                                              | 원인                                                             |
| --------------------------------------------------- | ---------------------------------------------------------------- |
| `not a Spec Kit project: no .specify/ directory found` | 초기화된 프로젝트 밖에서 실행                                  |
| `unknown artifact <name>`                           | 요청한 이름(그리고 주어진 경우 kind)과 일치하는 아티팩트가 없음   |
| `ambiguous artifact <name>: matches kinds [...]`    | 이름만으로는 둘 이상의 kind와 일치 — `--kind`로 다시 실행         |
| `artifact resolution failed`                        | 확장 레지스트리를 읽을 수 없거나, 오류로 인해 아티팩트 레이어 스택을 수집하지 못함 |

종료 코드 `2`는 사용법 오류(`--json` 플래그 누락 또는 유효하지 않은 `--kind` 값)용으로 예약되어 있으며, JSON 봉투 대신 일반 텍스트 메시지를 stderr에 출력합니다.
