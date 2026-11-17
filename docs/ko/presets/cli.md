# 프리셋 CLI 레퍼런스

모든 `specify preset` 명령에는 `specify init`으로 초기화된 프로젝트가 필요합니다.
그 프로젝트 안에서 실행하거나 `SPECIFY_INIT_DIR`을 프로젝트 루트로 설정하세요.
사용법은 `specify preset --help` 또는 하위 명령의 `--help`를 사용하세요.

## 사용 가능한 프리셋 검색

```bash
specify preset search [query]
```

| 옵션 | 설명 |
| --- | --- |
| `--tag <tag>` | 태그로 필터링 |
| `--author <author>` | 작성자로 필터링 |

활성 카탈로그를 검색합니다. 쿼리가 없으면 사용 가능한 모든 프리셋을 나열합니다.
탐색 전용 결과는 보이지만 ID로 설치할 수 없습니다.

## 프리셋 설치

```bash
specify preset add <preset_id>
specify preset add <preset_id> --version <version>
specify preset add --dev <path>
specify preset add --from <archive-url>
```

| 옵션 | 설명 |
| --- | --- |
| `--dev <path>` | 로컬 소스 디렉터리를 복사 |
| `--from <url>` | `.zip`, `.tar.gz`, `.tgz` 아카이브를 다운로드 |
| `--version <version>` | 이기는 카탈로그에서 정확한 릴리스를 선택. ID 설치 전용 |
| `--priority <N>` | 해석 우선순위. 양의 정수. 기본값 `10`, 낮을수록 우선 |

소스 형식은 하나만 고르세요. `--version`이 없으면 ID 설치는 가능한 경우 로컬에
번들된 프리셋을 먼저 사용하고, 그렇지 않으면 공개된 현재 카탈로그 릴리스를
사용합니다. 직접 URL 설치는 카탈로그를 참조하지 않습니다. 이미 설치된 프리셋은
다시 설치하기 전에 제거하거나 업데이트해야 합니다. `add`에는 `--force` 옵션이
없습니다.

`--version`은 프리셋 ID가 필요하며 `--from`이나 `--dev`와 결합할 수 없습니다.
요청한 릴리스가 이기는 카탈로그에 없으면 오류입니다. 더 낮은 우선순위의 카탈로그가
이를 제공할 수 없습니다. 탐색 전용 카탈로그는 어떤 릴리스도 설치할 수 없습니다.
버전 지정 카탈로그 설치는 설치된 프리셋을 수정하기 전에 아카이브의 `preset.yml`
ID와 버전을 검증합니다. 과거 릴리스에는 다운로드 시 검증되는 SHA-256 다이제스트가
필요합니다. 레거시 현재 릴리스는 이를 생략할 수 있습니다. 다운로드 URL이 없는
번들 릴리스는 일치하는 ID와 버전이 설치된 Spec Kit 패키지에 있을 때만 선택할 수
있습니다.

소스는 [`.presetignore`](developing.md#exclude-development-files-with-presetignore)를
존중하며 `.specify/presets/<id>/`로 복사됩니다.
프리셋의 명령 기여는 활성 통합에 대해 선택된 명령 또는 스킬 형식으로 등록됩니다.
generic 통합은 프리셋 오버라이드를 등록하지 않습니다. 필요한 확장 의존성이
충족되지 않으면 설치 실패나 자동 의존성 설치가 아니라 경고가 발생합니다.

다운로드 URL은 호스트 이름을 가진 HTTPS를 사용해야 합니다. HTTP는 `localhost`,
`127.0.0.1`, `::1`에만 허용됩니다. 아카이브는 루트 또는 하나의 최상위 디렉터리
안에 `preset.yml`을 담고 있어야 합니다.

## 프리셋 업데이트

```bash
specify preset update <preset_id> [--from <url>] [--dev <path>] [--priority <N>]
```

일반 `remove` 작업을 실행한 다음 일반 `add` 작업을 실행해 설치된 프리셋을
교체합니다. 명시적인 소스가 없으면 add는 평소의 번들 및 카탈로그 조회를
사용합니다. 이는 버전 비교나 제자리 패치가 아닙니다. `update`는 `--version`을
받지 않습니다.

> [!WARNING]
> 업데이트는 파괴적입니다. 제거가 성공하고 대체 설치가 실패하면 이전 프리셋은 이미
> 제거된 상태입니다. 소스 사전 점검, 스테이징, 롤백, 자동 수리, 복구 트랜잭션이
> 없습니다.

무언가를 제거하기 전에 명령은 `--from`과 `--dev`의 동시 사용, 빈 소스 인자, `1`
미만의 우선순위, 설치되지 않은 ID를 거부합니다. 이 검사 외에는 대체 소스를 미리
살펴보지 않습니다. `--from`이나 `--dev`를 쓰면 제거 전에 대체 매니페스트의 ID가
요청한 ID와 대조되지 않으므로, 다른 프리셋이 대신 설치될 수 있습니다.

add가 실패하면 명령은 대체 소스와 우선순위를 담은, 복사해 붙여 넣을 수 있는
`specify preset add` 재시도 명령을 보고합니다. Windows에서는 그 명령이
PowerShell용으로 형식화됩니다.

성공한 업데이트는 프리셋을 다시 활성화하고, `installed_at`을 다시 만들고, 설치된
프리셋 디렉터리 안의 로컬 수정을 제거합니다. 대체 우선순위는 기본값이 `10`입니다.
다른 값을 고르려면 `--priority`를 전달하세요. 명시적인 `--from`이나 `--dev`는
기록된 카탈로그 출처의 재사용이 아니라 의도적인 소스 변경입니다.

컨스티튜션 동기화가 활성화되어 있으면 일반 remove와 add의 조정 패스가 모두
실행됩니다. 대체가 실패하면 생성된 컨스티튜션이 제거된 프리셋이 없는 스택에 맞춰
조정된 상태로 남을 수 있습니다. 생성 파일 보호 장치는 여전히 손으로 편집한
컨스티튜션을 보호합니다. 최종 내용이 변경되지 않았을 때 다시 쓰기를 억제하는
업데이트 전용 최적화는 없습니다.

## 프리셋 제거

```bash
specify preset remove <preset_id>
```

설치된 프리셋과 그 레지스트리 항목을 삭제하고, 기록된 명령 및 스킬 출력을
정리하고, 영향받는 이름을 남은 스택에 맞춰 조정합니다. 제거는 그 프리셋이 기록한,
이전에 활성이었던 통합 디렉터리의 출력도 복원할 수 있습니다. 현재 통합의 파일을
삭제하는 데 한정되지 않습니다.

## 설치된 프리셋 나열

```bash
specify preset list
specify preset list --json
```

설치된 프리셋을 버전, 설명, 기여 수, 우선순위, 활성화 상태와 함께 나열합니다.
출력에는 비활성화된 프리셋이 포함되며 우선순위 순으로 정렬됩니다. 가장 낮은
우선순위 숫자가 먼저, 그다음 프리셋 ID의 알파벳순입니다.

`--json`은 정확히 하나의 JSON 배열을 stdout에 쓰고 성공 시 `0`으로 종료합니다.
모든 항목에는 다음 키가 있습니다.

| 키 | 값 |
| --- | --- |
| `id`, `name`, `description`, `version` | 프리셋 정체성과 메타데이터 |
| `author` | 작성자. 없으면 `null` |
| `priority` | 해석 우선순위 |
| `enabled` | 프리셋이 해석에 참여하는지 여부 |
| `source` | `{"kind":"local"}` 또는 `{"kind":"catalog","catalog":"<catalog-name>"}` |
| `provides` | `commands`, `templates`, `scripts`라는 이름의 수 |

로컬, 레거시, 형식이 잘못된 출처는 `{"kind":"local"}`로 보고됩니다.
`source` 객체는 프리셋이 어디에서 설치되었는지를 설명하며, 해석 스택에서의 기여
위치를 설명하지 않습니다.

옵션 파싱 이후의 런타임 실패는 성공 출력 없이 정확히 하나의 `{"error":"..."}`
객체를 stderr에 쓰고 `1`로 종료합니다. 원시 `--json` 토큰이 있는 사용법 오류도
stderr에 그 JSON 오류 형태를 사용하며, 보통 `2`인 사용법 종료 코드를 유지합니다.
`--json`이 없으면 도움말을 포함해 사람이 읽을 수 있는 동작은 바뀌지 않습니다.

## 프리셋 정보

```bash
specify preset info <preset_id>
specify preset info <preset_id> --versions
specify preset info <preset_id> --json
```

사람이 읽을 수 있는 출력은 설치된 프리셋을 먼저, 그렇지 않으면 사용 가능한
카탈로그 항목을 설명하며, 가능한 경우 메타데이터, 태그, 기여를 포함합니다.

`--versions`는 설치된 프리셋이 아니라 카탈로그를 참조합니다. 탐색 전용 항목이라도
공개된 현재 버전을 먼저, 이어서 과거 버전을 버전 내림차순으로 나열합니다. 버전을
나열한다고 해서 설치할 수 있게 되는 것은 아닙니다.

`--json`은 ID로 일치하는 설치된 프리셋 하나를 설명하고 JSON 객체 하나를 stdout에
씁니다. `list --json`과 같은 `id`, `name`, `description`, `version`, `author`,
`priority`, `enabled`, `source` 키를 갖습니다. `provides` 수 대신 `commands`,
`templates`, `scripts` 배열을 갖습니다. 각 배열은 매니페스트 선언 순서를 보존하며
다음을 담습니다.

| 키 | 값 |
| --- | --- |
| `name` | 논리적 기여 이름 |
| `description` | 설명. 없거나 비어 있으면 `""` |
| `source` | 기여하는 프리셋을 식별하는 `{"layer":"preset","presetId":"<id>"}` |
| `sourcePath` | 설치된 프리셋 기준 상대 경로인 매니페스트 `file` 경로 |
| `strategy` | `replace`, `prepend`, `append`, `wrap`. 생략되면 `replace` |

`--json`은 `--versions`와 결합할 수 없습니다(사용법 오류, 종료 코드 `2`).
카탈로그에만 있는 것을 포함해 설치되지 않은 프리셋이나 누락된 프로젝트는 stderr에
JSON 오류 객체 하나를 만들고 `1`로 종료합니다.
사용법 오류는 `list --json`과 마찬가지로 사용법 종료 코드를 유지합니다.

## 파일 해석

```bash
specify preset resolve spec-template
specify preset resolve speckit.specify
```

확장자 없는 템플릿 이름이나 점으로 구분된 명령 이름을 받습니다.
이름은 소문자, 숫자, 하이픈을 사용하며 명령 세그먼트는 비어 있지 않아야 합니다.
이 명령은 스크립트 유형 선택자를 제공하지 않습니다.

최상위 우선순위 레이어의 경로와 소스를 보여 줍니다. 출력이 합성된 것이면 유효
기반에서 시작하는 기여 합성 체인도 보여 줍니다. 표시되는 경로는 최종 합성 출력을
담은 파일이 아닙니다. 유효하지 않은 합성은 경고를 만들고, 없는 이름은 찾을 수
없다고 보고됩니다. 모든 아티팩트 종류와 숨겨진 레이어는 별도의
[아티팩트 레퍼런스](../overview/artifacts.md)를 사용하세요.

## 프리셋 활성화 또는 비활성화

```bash
specify preset enable <preset_id>
specify preset disable <preset_id>
```

설치된 프리셋이 이후의 해석에 참여하는지 여부를 바꿉니다.
현재 상태를 반복해도 상태 변경 없이 성공합니다.

이전에 등록된 명령과 스킬은 그대로 남습니다. 비활성화는 템플릿과 스크립트의
동작을 비교하는 데 유용하지만, 이미 전달된 오버라이드를 에이전트에서 거둬들이지는
않습니다. 그 출력이 더 이상 적용되지 않아야 한다면 제거를 사용하세요.
`constitution-sync`가 활성화되어 있으면 컨스티튜션 조정이 실행됩니다.

## 프리셋 우선순위 설정

```bash
specify preset set-priority <preset_id> <priority>
```

설치된 프리셋의 해석 우선순위를 바꿉니다. 값은 양의 정수여야 합니다. 숫자가
낮을수록 우선하며, 동률은 ID의 알파벳순으로 정해집니다. 이 명령은 레지스트리와,
참여를 선택한 컨스티튜션 조정을 갱신합니다. 명령 및 스킬 출력을 자체적으로 다시
스캐폴딩하지는 않습니다.

<a id="catalog-management"></a>

## 카탈로그 관리

```bash
specify preset catalog list
specify preset catalog add <url> --name <name>
specify preset catalog remove <name>
```

`list`는 활성 소스, 우선순위, 설치 권한을 보여 줍니다.
`add`는 프로젝트의 `.specify/preset-catalogs.yml`을 기록하고, `remove`는 그
프로젝트 설정에서 이름 있는 항목을 제거합니다.

| `catalog add` 옵션 | 설명 |
| --- | --- |
| `--name <name>` | 필수 카탈로그 이름 |
| `--priority <N>` | 기본값 `10`. 숫자가 낮을수록 먼저 확인됨 |
| `--install-allowed / --no-install-allowed` | ID 설치 허용. 기본값은 탐색 전용 |
| `--description <text>` | 선택적 설명 |

같은 이름의 카탈로그를 동일한 설정으로 다시 추가하면 설정을 다시 쓰지 않고
성공합니다. 같은 이름에 다른 설정은 거부됩니다. 먼저 항목을 제거하거나 다른
이름을 고르세요.

> [!IMPORTANT]
> 프로젝트의 `.specify/preset-catalogs.yml`은 여러분이 선택하지 않은 설치 허용
> 카탈로그를 선택할 수 있습니다. 낯선 프로젝트에서는 `catalog list`를 확인하세요.
> 직접 소유하거나 검증한 카탈로그에만 설치를 활성화하세요. `install_allowed`를
> 프리셋 코드가 검토되었다는 증거로 취급하지 마세요.

## 함께 보기

- [프리셋 개요](overview.md)
- [프리셋 개발하기](developing.md)
- [API 레퍼런스](api.md)
- [카탈로그 설정, 스키마, 신뢰](catalogs.md)
- [프리셋 게시하기](publishing.md)
