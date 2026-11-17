# 확장 CLI 레퍼런스

`specify extension`으로 확장을 관리하기 위한 전체 레퍼런스입니다.
모든 확장 명령에는 `specify init`으로 이미 초기화된 프로젝트가 필요합니다.

## 사용 가능한 확장 검색

```bash
specify extension search [query]
```

| 옵션         | 설명                                 |
| ------------ | ------------------------------------ |
| `--tag`      | 태그로 필터링                        |
| `--author`   | 작성자로 필터링                      |
| `--verified` | 검증된 확장만 표시                   |

모든 활성 카탈로그에서 쿼리와 일치하는 확장을 검색합니다. 쿼리가 없으면 사용 가능한 모든 확장을 나열합니다.

<a id="install-an-extension"></a>

## 확장 설치

```bash
specify extension add <name>
specify extension add --from <url>
specify extension add --dev <path>
```

| 옵션            | 설명                                                     |
| --------------- | -------------------------------------------------------- |
| `--dev`         | 로컬 디렉터리에서 설치(개발용)                           |
| `--from <url>`  | 카탈로그 대신 사용자 정의 URL에서 설치                   |
| `--version <v>` | 카탈로그가 공개한 정확한 버전을 설치                    |
| `--force`       | 확장이 이미 설치되어 있으면 덮어씀                       |
| `--priority <N>`| 해석 우선순위(기본값: 10. 낮을수록 우선순위가 높음) |

카탈로그, URL, 로컬 디렉터리로부터 확장을 설치합니다. 확장 명령은 활성 AI 코딩 에이전트 통합에 등록됩니다. `generic`의 경우 호출은 설정된 `--commands-dir`을 사용합니다. 기본적으로는 평면 명령 파일, `--skills`를 쓰면 `speckit-<name>/SKILL.md`입니다.

카탈로그 설치에는 위치 인자 이름이 필요하고, `--dev`에는 로컬 경로가 필요합니다.
`--from`을 쓰면 위치 인자 이름을 생략할 수 있습니다. 다운로드한 매니페스트가
설치되는 확장 ID를 제공합니다.

generic 통합의 새로 고침이 모든 확장 호출을 만들어 낼 수 없으면(예: 명령이나 스킬이 사용자에 의해 수정되었거나 그 소스가 없는 경우), 경고하고 그 확장의 이전 등록 아티팩트를 복원합니다. 다른 확장은 여전히 새로 고칠 수 있습니다.

버전을 지정하지 않은 카탈로그 설치는 여전히 공개된 현재 버전을 선택합니다.
`--version`은 그 확장 ID에 대해 이기는 카탈로그 소스만 사용합니다. 요청한 버전이
없을 때 더 낮은 우선순위의 소스로 폴백하지 않습니다.
탐색 전용 카탈로그는 설치할 수 없는 상태로 남습니다. `--version`은 `--dev`나 직접
URL인 `--from` 옵션과 결합할 수 없습니다. `--version` 카탈로그 설치의 경우,
다운로드한 아카이브의 확장 ID와 버전이 설치 전에 요청한 카탈로그 ID 및 버전과
비교됩니다. 버전을 지정하지 않은 카탈로그 설치와 직접 `--from` 설치는
매니페스트를 검증하지만 그 ID나 버전을 예상되는 카탈로그 값과 비교하지 않습니다.

## 확장 제거

```bash
specify extension remove <name>
```

| 옵션            | 설명                                           |
| --------------- | ---------------------------------------------- |
| `--keep-config` | 제거 중 설정 파일 보존                         |
| `--force`       | 확인 프롬프트 건너뛰기                         |

설치된 확장을 제거합니다. 설정 파일은 기본적으로 백업됩니다. 그대로 두려면 `--keep-config`를, 확인을 건너뛰려면 `--force`를 사용하세요.

<a id="list-installed-extensions"></a>

## 설치된 확장 나열

```bash
specify extension list
specify extension list --json
```

| 옵션          | 설명                                               |
| ------------- | -------------------------------------------------- |
| `--available` | 설치된 확장과 설치 힌트를 표시. 카탈로그를 조회하지 않음 |
| `--all`       | 현재는 `--available`과 같은 동작                    |
| `--json`      | 설치된 확장을 JSON으로 출력                        |

설치된 확장을 상태, 버전, 명령 수와 함께 나열합니다.

도움말은 `--available`과 `--all`이 카탈로그 항목을 포함한다고 설명하지만, 현재
핸들러는 설치되지 않은 확장을 가져오거나 나열하지 않습니다. 사용 가능한 확장을
탐색하려면 `specify extension search`를 사용하세요.

`--json`은 JSON 배열을 stdout에 씁니다. 모든 항목에는 `id`, `name`,
`description`, `version`, `author`, `priority`, `enabled`, `source`, `provides`
키가 있습니다. `author`는 없으면 `null`입니다. `source`는 로컬, 레거시, 형식이
잘못된 출처에 대해서는 `{"kind":"local"}`이고, 유효한 카탈로그 소스에 대해서는
`{"kind":"catalog","catalog":"<catalog-name>"}`입니다.
확장의 `provides`에는 `commands`, `templates`, `scripts`, `hooks` 수가 들어
있습니다. `--available`과 `--all`은 JSON 출력을 설치된 확장 너머로 넓히지
않습니다. 성공 시 `--json`은 정확히 하나의 배열을 stdout에 쓰고 0으로 종료합니다.
옵션 파싱 이후의 런타임 실패는 정확히 하나의 `{"error":"..."}` 객체를 stderr에
쓰고 1로 종료합니다. 파싱이 사용법 오류를 일으키고 원시 `--json` 토큰이 있으면, 그
JSON 오류 객체를 stderr에 쓰고 사용법 종료 코드(보통 2)를 유지합니다. `--json`이
없으면 도움말을 포함해 기존의 사람이 읽을 수 있는 동작은 바뀌지 않습니다.

## 확장 정보

```bash
specify extension info <name>
specify extension info <name> --versions
specify extension info <name> --json
```

설명, 버전, 명령, 설정을 포함해 설치되었거나 사용 가능한 확장에 대한 상세 정보를 보여 줍니다.
`--versions`는 이기는 카탈로그 소스가 공개한 현재 및 과거 버전을 나열하며, 탐색
전용 소스를 설치 불가로 표시합니다.
동등한 PEP 440 버전 표기(예: `v1.0`과 `1.0`)는 같은 릴리스를 선택합니다.
카탈로그가 공개한 표기는 그대로 보입니다.

이전 버전을 공개하는 데 쓰는 카탈로그 형식과 번들 고정 동작은
[과거 릴리스](catalogs.md#historical-releases)를 참고하세요.

`--json`은 설치된 확장 하나를 설명하고 단일 JSON 객체를 stdout에 씁니다. `<name>`은
`--json`이 없을 때와 같이 매칭됩니다. 확장 ID가 먼저이고, 그다음 대소문자를
무시한 고유한 표시 이름입니다. 객체는 일치하는 `specify extension list --json`
항목과 같은 `id`, `name`, `description`, `version`, `author`, `priority`,
`enabled`, `source` 키를 갖습니다. `provides` 수 대신 `commands`, `templates`,
`scripts`, `hooks` 배열을 갖습니다. 명령, 템플릿, 스크립트 항목에는 `name`,
`description`(없으면 `""`), `source`(`{"layer": "extension", "extensionId": "<id>"}`,
즉 그것을 제공하는 확장. 최상위 `source`는 어디에서 설치되었는지를 나타냄),
`sourcePath`(확장 디렉터리 기준 상대 경로인 매니페스트 `file`)가 있습니다. 스크립트
항목에는 매니페스트가 선언한 경우 `runtimes`도 있습니다. 확장 항목에는
`strategy`가 없습니다. 확장이 제공하는 파일은 항상 대체하기 때문입니다. 각 훅
항목에는 `trigger`(`after_tasks` 같은 훅 이벤트), `targetCommand`,
`optional`(기본값 `true`), `priority`(기본값 `10`)가 있습니다. 한 이벤트가 같은
명령을 두 번 이상 선언하면 훅이 등록될 때와 마찬가지로 마지막 선언이 이깁니다.
`--json`은 `--versions`와 결합할 수 없습니다(사용법 오류, 종료 코드 2).
설치되지 않은 확장, 모호한 이름, 누락된 프로젝트는 `{"error":"..."}` 객체 하나를
stderr에 쓰고 1로 종료합니다. 사용법 오류는 `list --json`과 마찬가지로 종료
코드(보통 2)를 유지합니다.

<a id="update-extensions"></a>

## 확장 업데이트

```bash
specify extension update [<name>]
```

특정 확장을, 또는 이름이 없으면 설치된 모든 확장을 업데이트합니다.

번들된 확장(`agent-context`, `git` 등)에는 다운로드 URL이 없습니다. 그 업데이트는 실행 중인 spec-kit 릴리스와 함께 제공된 사본으로부터 설치됩니다. 카탈로그가 사용 중인 spec-kit 릴리스가 제공하는 것보다 더 새로운 버전을 공개하면, 업데이트는 먼저 spec-kit 업그레이드가 필요하다고 보고됩니다.

`generic`의 경우, 실패한 업데이트는 다른 통합이 현재 활성 상태이더라도 현재 위치뿐 아니라 이전에 설정된 `--commands-dir` 위치에서도 해시로 소유한 호출을 복원합니다.

## 확장 활성화 / 비활성화

```bash
specify extension enable <name>
specify extension disable <name>
```

제거하지 않고 확장을 비활성화합니다. 비활성화된 확장은 레지스트리 기반 아티팩트
해석과 향후의 통합 등록에서 제외되고, 그 선언적 훅은 `.specify/extensions.yml`에서
비활성으로 표시됩니다.
네이티브 런타임 이벤트 설정은 이런 상태 변경 후에 최선의 노력으로 새로
고쳐집니다. 새로 고침이 실패하면 명령은 경고하지만, 오래된 네이티브 훅이 활성
상태로 남아 여전히 실행될 수 있습니다. 영향받는 통합의 새로 고침을
`specify integration upgrade <key>`로 다시 시도하세요.

generic이 아닌 통합은 통합 전환, 제거, 조정이 그것을 제거할 때까지 호출 가능한
상태로 남는 생성된 명령 또는 스킬 파일을 유지할 수 있습니다. 비활성화는 보편적인
실행 차단이 아닙니다. 훅 전용 확장은 generic 명령 출력 설정이 없거나 유효하지
않아도 설치, 활성화, 비활성화할 수 있습니다. 명령이 있는 확장은 여전히 유효한
설정이 필요합니다. `enable`로 다시 활성화하세요.

`generic`의 경우 비활성화는 출력 디렉터리가 옮겨진 뒤에도 해시로 소유한 호출을 제거하지만, 새 디렉터리에 있는 관련 없는 같은 이름의 파일은 보존합니다.

<a id="set-extension-priority"></a>

## 확장 우선순위 설정

```bash
specify extension set-priority <name> <priority>
```

확장의 아티팩트 해석 우선순위를 바꿉니다. 템플릿과 스크립트 같은 기여 아티팩트에
대해 숫자가 낮을수록 우선하며, 동률은 확장 ID로 정렬됩니다. 설치는 설치된 확장과의
정확한 명령 이름 또는 별칭 충돌, 그리고 코어 명령과의 정규화된 출력 이름 충돌을
거부합니다. 우선순위가 중복된 명령 이름의 설치를 허용하지는 않습니다.

<a id="catalog-management"></a>

## 카탈로그 관리

확장 카탈로그는 `search`와 `add`가 확장을 찾는 위치를 제어합니다. 카탈로그는 우선순위 순으로 확인됩니다(숫자가 낮을수록 우선순위가 높음).

탐색 전용 카탈로그는 검색할 수 있지만 설치 소스로 쓸 수 없습니다. 직접 소유하고
검증한 소스에만 설치를 활성화하세요. 서드파티 확장을 설치하기 전에
[카탈로그 신뢰 모델](catalogs.md#trust-model-discovery-only-vs-install-sources)을
참고하세요.

### 카탈로그 나열

```bash
specify extension catalog list
```

스택의 모든 활성 카탈로그를 우선순위 및 설치 권한과 함께 보여 줍니다.

### 카탈로그 추가

```bash
specify extension catalog add <url>
```

| 옵션                                 | 설명                                               |
| ------------------------------------ | -------------------------------------------------- |
| `--name <name>`                      | 필수. 카탈로그의 고유한 이름                       |
| `--priority <N>`                     | 우선순위(기본값: 10. 낮을수록 우선순위가 높음)     |
| `--install-allowed / --no-install-allowed` | 카탈로그를 신뢰할 수 있는 설치 소스로 표시. 직접 소유하고 검증한 카탈로그에만 활성화하고, 탐색 전용 소스에는 꺼 두세요(기본값). 검증되지 않은 공개 카탈로그에는 절대 활성화하지 마세요. |
| `--description <text>`               | 선택적 설명                                        |

카탈로그를 프로젝트의 `.specify/extension-catalogs.yml`에 추가합니다.

같은 이름의 카탈로그를 동일한 설정으로 다시 추가하면 설정 변경 없이 성공합니다. 다른 설정은 거부됩니다.

### 카탈로그 제거

```bash
specify extension catalog remove <name>
```

프로젝트 설정에서 카탈로그를 제거합니다.

## 함께 보기

- [확장 사용자 가이드](overview.md)
- [확장 카탈로그](catalogs.md)
- [확장 API 레퍼런스](api.md)
- [확장 개발하기](developing.md)
- [확장 게시하기](publishing.md)
- [커뮤니티 확장](../community/extensions.md)
