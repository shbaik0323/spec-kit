# 워크플로 오버레이

워크플로 오버레이를 사용하면 프로젝트가 설치된 `workflow.yml`을 편집하지 않고 설치된 워크플로를 확장하거나 재정의할 수 있습니다. 덕분에 `specify bundle update`나 `specify workflow add` 업그레이드를 거쳐도 로컬 사용자 정의가 안전하게 유지됩니다.

`specify workflow run <workflow-id>`가 워크플로를 로드할 때, 엔진은 기반 워크플로를 그 워크플로 id에 대해 활성화된 모든 오버레이와 합성합니다. 결과는 다른 워크플로 정의와 마찬가지로 검증됩니다.

## 오버레이의 동작 방식

오버레이는 기반 워크플로의 스텝 목록에 대한 편집 작업의 집합을 선언하는 YAML 파일입니다. 오버레이는 낮은 쪽이 이기는 우선순위를 사용합니다. 우선순위 숫자가 높은 것이 먼저, 낮은 것이 마지막에 적용됩니다. 우선순위가 같은 오버레이는 ID의 알파벳순으로 적용되며 마지막 ID가 충돌에서 이깁니다.

프로젝트 오버레이 파일은 다음 위치에 있습니다.

| 위치 | 용도 |
| --- | --- |
| `.specify/workflows/overlays/<id>/*.yml` | 프로젝트 로컬 사용자 정의 |

## 오버레이 파일 형식

권장되는 편집 형식은 작업 이름을 키로, 앵커 스텝 id를 값으로 사용합니다.

```yaml
id: "my-overlay"
extends: "speckit"
priority: 10
enabled: true
edits:
  - insert_after: implement
    step:
      id: run-lint
      type: shell
      run: "ruff check src/"

  - replace: review-spec
    step:
      id: review-spec
      type: gate
      message: "Review the generated spec (overlay override)."
      options: [approve, reject]
      on_reject: abort
```

명시적인 형식도 지원됩니다.

```yaml
edits:
  - operation: insert_after
    anchor: implement
    step:
      id: run-lint
      type: shell
      run: "ruff check src/"
```

### 필드

| 필드 | 필수 | 설명 |
| --- | --- | --- |
| `id` | 예 | 이 오버레이의 식별자. `specify workflow overlay *` 명령에서 사용됩니다. 소문자, 숫자, 하이픈만 가능하며 점, 밑줄, 경로 구분자, `overlays`는 쓸 수 없습니다. |
| `extends` | 예 | 이 오버레이가 적용되는 워크플로 id. `id`와 같은 안전한 id 형식을 사용하며 `overlays`, `runs`, `steps`는 예약되어 있습니다. |
| `priority` | 아니요 | 정수. 기본값은 `10`. 값이 낮을수록 우선순위가 높고 충돌에서 이깁니다. 없거나 유효하지 않은 값은 `10`으로 폴백합니다. |
| `enabled` | 아니요 | 불리언. 기본값은 `true`. 비활성화된 오버레이는 무시됩니다. |
| `edits` | 예 | 비어 있지 않은 편집 작업 목록. |

### 편집 작업

| 작업 | `step` 필요 | 효과 |
| --- | --- | --- |
| `insert_after` | 예 | 앵커 스텝 바로 뒤에 `step`을 삽입합니다. |
| `insert_before` | 예 | 앵커 스텝 바로 앞에 `step`을 삽입합니다. |
| `replace` | 예 | 앵커 스텝을 `step`으로 교체합니다. |
| `remove` | 아니요 | 앵커 스텝을 목록에서 제거합니다. |

`anchor`는 기반 워크플로에 있는 스텝의 `id`입니다. 앵커는 `then`, `else`, `steps`, `cases.*`, `default` 블록 안에서 재귀적으로 해석되므로 중첩된 기반 스텝도 대상으로 삼을 수 있습니다. 팬아웃 템플릿(`fan-out` 스텝 안의 `step`)은 유효한 앵커가 **아닙니다**.

스텝 id에는 `:`가 들어갈 수 없습니다. 이 문자는 엔진이 생성하는 중첩 id용으로 예약되어 있습니다.

## 오버레이 관리

오버레이를 추가, 나열, 우선순위 지정, 활성화, 비활성화, 제거하려면 [오버레이 CLI 명령](cli.md#overlay-management)을 사용하고, 합성 결과는 `specify workflow resolve <workflow-id>`로 확인하세요.

## 예시: 구현 후 자동 린트 추가

내장 `speckit` 워크플로가 있을 때 `project-overlay.yml`을 만듭니다.

```yaml
id: "add-lint"
extends: "speckit"
priority: 10
edits:
  - insert_after: implement
    step:
      id: run-lint
      type: shell
      run: "ruff check src/"
```

설치합니다.

```bash
specify workflow overlay add project-overlay.yml --priority 10
```

워크플로를 실행합니다.

```bash
specify workflow run speckit -i spec="Build a kanban board"
```

합성된 워크플로는 이제 전체 SDD 주기를 실행하고 `implement` 스텝 뒤에 `ruff check src/`를 자동으로 실행합니다.

## 예시: 게이트 교체

```yaml
id: "skip-plan-review"
extends: "speckit"
priority: 5
edits:
  - replace: review-plan
    step:
      id: review-plan
      type: command
      command: speckit.plan
      input:
        args: "{{ inputs.spec }}"
```

우선순위 값이 낮을수록 우선순위가 높습니다. 이 오버레이의 `priority: 5`는 우선순위
10인 오버레이와의 충돌에서 이깁니다. `review-plan` 게이트를 또 다른 plan 명령으로
교체하므로, 기존 plan 명령도 여전히 실행됩니다.

<a id="workflow-slots-upstream-extension-points"></a>

## 워크플로 슬롯(업스트림 확장 지점)

워크플로 작성자는 `type: slot`으로 이름 있는, 아무 동작도 하지 않는 워크플로 슬롯을
선언할 수 있습니다.

```yaml
- id: post-implement
  type: slot
  name: "Post-implementation checks"
```

스텝 `id`가 고유한 오버레이 앵커이고, `name`은 비어 있지 않아야 하는 사람이 읽을
수 있는 레이블일 뿐입니다. 채워지지 않은 슬롯은 `output: {slot: <name>}`을 가진
`skipped` 스텝으로 완료되므로 이후 스텝은 정상적으로 계속됩니다.

슬롯은 그 `name`이 아니라 스텝 `id`에 고정된, 스키마에 맞는 오버레이 `replace`
편집으로 채우세요.

```yaml
id: fill-post-implement
extends: my-workflow
edits:
  - replace: post-implement
    step:
      id: post-implement
      type: shell
      run: "echo Run project-specific checks"
```

이후의 표현식이나 `fan-in.wait_for`가 슬롯을 참조한다면 슬롯의 `id`를 재사용하세요.
대체 스텝은 그 이후 스텝이 소비하는 모든 출력 키도 보존해야 합니다. 채워지지 않은
워크플로 슬롯은 `steps.<id>.output.slot`만 제공합니다. 런타임에 증식되는 템플릿은
오버레이 앵커가 될 수 없으므로, 슬롯 스텝은 `fan-out.step` 템플릿 안에서 지원되지
않습니다.

## 번들 및 업데이트와의 상호 작용

`specify workflow add <local-directory>`는 로컬 워크플로 패키지 전체를
`.specify/workflows/<id>/`에 설치합니다. 아카이브 설치도 같은 패키지 내용을
보존합니다.

설치된 워크플로가 새로 고쳐지거나 재설치될 때, `.specify/workflows/overlays/<id>/`의 프로젝트 오버레이는 설치된 워크플로 디렉터리 밖에 있으므로 보존됩니다.

## 제한 사항

- 오버레이는 스텝 목록에만 작용합니다. 워크플로 메타데이터, 입력, 출력,
  `requires`, 워크플로 수준 기본값은 바꿀 수 없습니다. 스텝의 표현식을 바꾸려면 그
  스텝을 교체하세요.
- 팬아웃 템플릿은 앵커로 사용할 수 없습니다.
- 기반 워크플로에 존재하지 않는 스텝 id를 대상으로 하는 오버레이는 워크플로가 해석될 때 검증 오류를 발생시킵니다.
- 오버레이는 다른 오버레이가 추가한 스텝을 대상으로 삼을 수 없습니다.
- 오버레이는 새 입력을 추가하거나 기반 워크플로의 입력 스키마를 바꿀 수 없습니다.

## 함께 보기

- [워크플로 CLI](cli.md#overlay-management)
- [스텝 타입](../workflow-steps/types.md)
