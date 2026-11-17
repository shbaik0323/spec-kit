# 통합 CLI 레퍼런스

이 레퍼런스는 공통 `specify integration` 명령, 카탈로그 관리, 라이프사이클 동작을
문서화합니다. 에이전트별 설정, 설치 경로, 옵션, 제한 사항은
[통합 가이드](index.md)부터 시작하세요.

이 명령들은 `specify init`으로 초기화된 프로젝트 안에서 실행하세요.
새 프로젝트에는 `specify init <project> --integration <key>`를 사용하세요.

## 지원되는 AI 코딩 에이전트

[통합 목록](index.md#choose-an-integration)이 지원되는 에이전트와 정확한 키를
나열하며, 통합마다 전용 가이드가 있습니다. 로컬 사용 가능 여부를 확인하려면
`specify integration list`를 실행하세요.

<a id="command-invocation"></a>

## 명령 호출

Spec Kit의 프로세스 단계는 터미널이 아니라 코딩 에이전트 안에서 호출하세요.
GitHub Copilot의 기본 모드에서 이들은 에이전트의 채팅에서 호출하는 스킬입니다.
CLI 설치와 설정 명령은 별개의 터미널 작업입니다. 단계는 통합 전반에서 같지만,
표기는 에이전트와 설치된 모드에 따라 달라집니다.

| 통합 또는 모드 | SDD 예시 | 확장 예시 |
| --- | --- | --- |
| GitHub Copilot, 기본 스킬 모드 | `/speckit-specify` | `/speckit-bug-assess` |
| 레퍼런스에서 쓰는 점 표기 슬래시 명령 | `/speckit.specify` | `/speckit.bug.assess` |
| Codex, Command Code, ZCode 스킬 | `$speckit-specify` | `$speckit-bug-assess` |
| Kimi 스킬 | `/skill:speckit-specify` | `/skill:speckit-bug-assess` |

에이전트가 노출하는 형식을 사용하세요. 스킬 및 명령 레이아웃은
[GitHub Copilot 가이드](copilot.md)를 참고하거나, [통합 목록](index.md)에서 다른
에이전트를 선택하세요.

## 사용 가능한 통합 나열

```bash
specify integration list
```

| 옵션        | 설명                                                                                                                    |
| ----------- | ----------------------------------------------------------------------------------------------------------------------- |
| `--catalog` | 이 프로젝트에 설치되지 않은 외부 통합을 포함해 카탈로그도 둘러봅니다. |

내장 통합과 신뢰된 설치 외부 통합, 그 설정 요구 사항, 다중 설치 안전성 선언을
보여 줍니다. 설치된 통합과 기본값은 따로 표시됩니다.
`CLI Required` 열은 통합이 IDE 전용인지가 아니라 `config.requires_cli`가 선언한
에이전트 CLI 요구 사항을 보고합니다. 외부 어댑터는 디스크립터의 `requires.tools`를
통해 실행 파일을 별도로 요구할 수 있습니다. `CLI Required`가 `no`여도 설치와
`specify check`는 그 도구 요구 사항을 검증합니다.

<a id="search-available-integrations"></a>

## 사용 가능한 통합 검색

```bash
specify integration search [query]
```

| 옵션       | 설명               |
| ---------- | ------------------ |
| `--tag`    | 태그로 필터링      |
| `--author` | 작성자로 필터링    |

활성 카탈로그 스택을 검색합니다. 쿼리가 없으면 사용 가능한 모든 통합을 나열합니다.

## 통합 정보

```bash
specify integration info <integration_id>
specify integration info <integration_id> --versions
```

통합의 카탈로그 메타데이터, 소스 카탈로그, 활성 여부를 보여 줍니다.
`--versions`는 이기는 카탈로그가 공개한 현재 및 과거 버전을 나열하며, 현재
릴리스가 맨 먼저 오고 이전 릴리스는 버전 내림차순으로 옵니다. 탐색 전용 소스는
그렇게 표시됩니다.

<a id="install-an-integration"></a>

## 통합 설치

```bash
specify integration install <key>
```

| 옵션                     | 설명                                                                     |
| ------------------------ | ------------------------------------------------------------------------ |
| `--script sh\|ps\|py`    | 스크립트 유형: `sh`(bash/zsh), `ps`(PowerShell), `py`(Python)            |
| `--force`                | 다중 설치 안전으로 선언되지 않은 통합과 나란히 설치하는 데 동의 |
| `--integration-options`  | 통합별 옵션(예: `--integration-options="--commands-dir .myagent/cmds"`) |
| `--trust-integration` | 코드를 검토한 뒤, 대화형 신뢰 프롬프트 없이 외부 어댑터의 Python 실행을 사전 승인 |

통합을 현재 프로젝트에 설치합니다. 다중 설치는 관련된 모든 통합이 안전하다고
선언된 경우에만 자동으로 진행됩니다. 그렇지 않으면 `switch`로 기본값을 교체하거나
`--force`로 다중 설치에 동의하세요. 설정이 실패하면 롤백이 트리거됩니다.

**카탈로그 이력은 메타데이터일 뿐입니다.** `integration install`은 여전히 등록된
내장 구현을 직접 해석하거나, 설치가 활성화된 카탈로그의 현재 외부 어댑터 릴리스를
해석하며, 과거 카탈로그 기록은 사용하지 않습니다. `integration install --version`은
없습니다. 기본 커뮤니티 카탈로그는 탐색 전용으로 남습니다.

추가 통합을 설치해도 기본 통합은 바뀌지 않습니다. 기본값을 바꾸려면 `specify integration use <key>`를 사용하세요.

확장과 프리셋은 기본 통합만 따릅니다. 다른 통합을 활성화하고 그 통합에 명령을
등록하려면 `use <key>`나 `switch <key>`를 사용하세요.

<a id="catalog-installed-external-adapters"></a>

### 카탈로그로 설치하는 외부 어댑터

초기화된 프로젝트에 검토한 카탈로그를 등록한 다음 그 어댑터를 설치합니다.

```bash
specify integration catalog add https://example.com/catalog.json --name samples
specify integration install sample-agent
specify integration use sample-agent
```

설치는 Python을 다운로드/임포트하기 **전에** 신뢰 여부를 묻습니다. 자동화에서는
검토한 코드를 명시적으로 승인하세요.

```bash
specify integration install sample-agent --trust-integration
specify integration upgrade sample-agent --trust-integration
```

소스에는 `install_allowed: true`가 있어야 합니다. 기본 커뮤니티 카탈로그는 탐색
전용이며, `--force`도 신뢰 플래그도 그 정책을 우회하지 않습니다.
카탈로그 목록/검색/정보는 카탈로그 코드를 임포트하지 않습니다. 필수 메타데이터와
지원되는 아카이브 URL은 [카탈로그 스키마](catalog.md#catalog-schema)를 참고하세요.
검색은 설치가 활성화된 소스에 대해 `specify integration install <id>`를
안내합니다. 탐색 전용 결과는 설치를 안내하지 않습니다.

`specify check`는 외부 어댑터의 카탈로그 ID가 아니라 디스크립터의 필수 도구를
확인합니다. 선언된 필수 도구가 없으면 CLI 어댑터는 런타임 실행 파일을 사용해
확인됩니다. 선택적 도구는 도구 누락 오류를 만들지 않습니다. 이 검사는 실행 파일을
실행하거나 범용 버전 탐침을 호출하지 않습니다.

설치된 어댑터 코드는 생성된 에이전트 파일과 별도로 유지됩니다. 새 CLI 프로세스는
카탈로그를 참조하지 않고 신뢰된 패키지를 로드합니다. 코드가 없거나, 수정되었거나,
호환되지 않으면 조용한 폴백이 아니라 오류입니다. 업그레이드는 카탈로그의 현재
버전을 가져오고, 제거는 기본적으로 수정된 생성 파일을 보존하면서 패키지를
제거합니다.
메타데이터 전용인 `workflow status`와 `workflow info`는 설치된 어댑터가 손상된
경우를 포함해 어댑터 코드를 로드하지 않고도 계속 사용할 수 있습니다. 워크플로
실행과 재개는 여전히 어댑터 로드 실패를 명시적으로 보고합니다.

실행 동의는 사용자 로컬이며 프로젝트, 통합, 검증된 패키지 내용에 묶입니다.
프로젝트 메타데이터를 복사해도 동의는 옮겨지지 않습니다. 프로젝트를 복사하거나
사용자를 바꾼 뒤에는 어댑터를 검토하고 설치가 활성화된 카탈로그로부터
재승인하세요.

```bash
specify integration upgrade sample-agent --force --trust-integration
```

강제 제거는 패키지 코드가 신뢰되지 않거나 그 디렉터리 전체가 없을 때도 동작하며,
그 코드를 임포트하지 않습니다.

초기화 시에는 프로젝트/사용자 카탈로그나 `SPECKIT_INTEGRATION_CATALOG_URL`이
외부 어댑터를 제공할 수 있습니다.

```bash
specify init my-project --integration sample-agent --trust-integration
```

패키지를 게시하기 전에 [외부 어댑터 API](api.md)를 검토하세요. 호스트 API와 표준
라이브러리를 사용하는 어댑터에는 pip 설치나 소스 레지스트리 편집이 필요하지
않습니다.

CLI가 다중 설치를 거부하면 `specify version`을 확인하고
[업그레이드 가이드](../upgrade.md)를 따르세요. 일회성 `uvx` 호출은 `PATH`에 있는
영구 `specify` 실행 파일을 업데이트하지 않습니다.

<a id="uninstall-an-integration"></a>

## 통합 제거

```bash
specify integration uninstall [<key>]
```

| 옵션      | 설명                                                |
| --------- | --------------------------------------------------- |
| `--force` | 파일이 수정되었더라도 제거                           |

지정한 통합을 제거하며, 키를 주지 않으면 기본 통합을 제거합니다.
매니페스트로 추적되는 생성 파일은 정리에 콘텐츠 해시를 사용합니다.

- **수정되지 않은 파일**은 자동으로 제거됩니다.
- **수정된 파일**(수동으로 편집한 파일)은 사용자 정의가 사라지지 않도록 보존됩니다.
- 수정된 추적 파일도 제거하려면 `--force`를 사용하세요.

이것이 매니페스트로 추적되는 생성 파일에 대한 규칙입니다. 통합별 정리는 다를 수
있습니다. [Hermes 전역 스킬](hermes.md#global-ownership-and-removal)과
[Kimi 레거시 스킬](kimi.md#legacy-cleanup-on-uninstall)은 수정 파일 보호 없이
제거될 수 있습니다. 제거하거나 다른 통합으로 전환하기 전에 에이전트 가이드를
확인하세요.

제거는 통합에 등록된 확장 및 프리셋 아티팩트도 제거합니다. 확장과 프리셋 자체는
설치된 상태로 남습니다. 기본 통합을 제거하면 남아 있는 통합이 선택되고 활성화된
기여가 그 통합에 등록됩니다. 마지막 통합을 제거해도 `none`이 자동으로 설치되지는
않습니다.

<a id="switch-to-a-different-integration"></a>

## 다른 통합으로 전환

```bash
specify integration switch <key>
```

| 옵션                     | 설명                                                                     |
| ------------------------ | ------------------------------------------------------------------------ |
| `--script sh\|ps\|py`    | 스크립트 유형: `sh`(bash/zsh), `ps`(PowerShell), `py`(Python)            |
| `--force`                | 제거 중 수정된 파일을 강제로 삭제. 대상이 이미 설치되어 있으면 사용자 정의된 공유 인프라를 덮어씀 |
| `--refresh-shared-infra` | 설치된 대상과 현재 기본값을 포함해, 사용자 정의된 공유 스크립트, 페이지 템플릿, `.specify/.gitignore`를 덮어씀 |
| `--integration-options`  | 대상 통합이 아직 설치되지 않은 경우의 옵션                               |
| `--trust-integration` | 대화형 신뢰 프롬프트 없이, 검토한 외부 어댑터의 다운로드와 실행을 승인 |

- **대상이 설치되지 않음:** 현재 기본값을 제거하고, 대상을 설치하고, 그것을
  기본값으로 만듭니다. 설치된 다른 통합은 남습니다.
- **대상이 이미 설치됨:** `use`처럼 어떤 통합도 제거하지 않고 기본값을 바꿉니다.
  설정 옵션을 바꾸려면 먼저 `upgrade <key> --integration-options ...`를 실행한
  다음 `use <key>`를 실행하세요. `switch`는 설치된 대상에 대한 설정 옵션을
  거부합니다.

두 경로 모두 새 기본값에 활성화된 확장과 프리셋을 등록합니다.
대상이 이미 현재 기본값이면 플래그 없는 호출은 아무것도 하지 않습니다.
`--refresh-shared-infra`나 `--force`는 통합을 재설치하거나 확장과 프리셋을 다시
등록하지 않고 공유 인프라만 새로 고칩니다.

설치되지 않은 대상의 경우 `--force`는 나가는 통합의 편집된 파일 제거를 제어하고,
`--refresh-shared-infra`만 공유 사용자 정의의 덮어쓰기를 허용합니다. 설치된 대상의
경우 두 플래그 모두 그 공유 덮어쓰기를 허용합니다. 심볼릭 링크된 공유 경로는 어느
플래그를 써도 보호된 채로 남습니다.

다운로드하고 설치해야 하는 외부 어댑터로 전환할 때는 코드를 검토한 뒤에만
`--trust-integration`을 사용하세요.

```bash
specify integration switch sample-agent --trust-integration
```

이미 설치된 어댑터로 전환할 때도 기존 신뢰를 검증합니다. 이 플래그는 그 검증이나
카탈로그의 탐색 전용 정책을 우회하지 않습니다. 신뢰 요구 사항은
[카탈로그로 설치하는 외부 어댑터](#catalog-installed-external-adapters)를
참고하세요.

<a id="use-an-installed-integration"></a>

## 설치된 통합 사용

```bash
specify integration use <key>
```

| 옵션      | 설명                                                |
| --------- | --------------------------------------------------- |
| `--force` | 기본값을 바꾸면서 관리되는 공유 템플릿을 덮어씀      |

설치된 다른 통합을 제거하지 않고 기본 통합을 설정합니다. 명령 참조가 새 기본 통합의 호출 방식과 일치하도록 관리되는 공유 템플릿도 새로 고칩니다. 수정되었거나 추적되지 않는 공유 템플릿은 `--force`를 쓰지 않는 한 보존됩니다.

또한 다른 통합이 활성 상태일 때 설치된 기여를 포함해, 활성화된 확장 및 프리셋의
명령이나 스킬을 새 기본값에 등록합니다.

<a id="upgrade-an-integration"></a>

## 통합 업그레이드

```bash
specify integration upgrade [<key>]
```

| 옵션                     | 설명                                                                     |
| ------------------------ | ------------------------------------------------------------------------ |
| `--force`                | 파일이 수정되었더라도 덮어씀                                             |
| `--script sh\|ps\|py`    | 스크립트 유형: `sh`(bash/zsh), `ps`(PowerShell), `py`(Python)            |
| `--integration-options`  | 통합에 대한 옵션                                                         |
| `--trust-integration`    | 검토한 대체 외부 어댑터의 다운로드/임포트를 승인                         |

설치된 통합을 갱신된 템플릿과 명령으로 재설치합니다(예: Spec Kit 업그레이드 후). 기본값은 기본 통합입니다. 키를 제공하면 설치된 통합 중 하나여야 합니다. 로컬에서 수정된 파일을 감지하고 `--force`를 쓰지 않는 한 업그레이드를 차단합니다. 이전 설치에서 남았지만 더는 필요 없는 오래된 파일은 자동으로 제거됩니다. 공유 템플릿은 기본이 아닌 통합을 업그레이드할 때도 기본 통합에 맞춰진 상태를 유지합니다.

활성화된 확장과 프리셋은 현재 활성(기본) 통합을 업그레이드할 때만 다시 등록됩니다. 기본이 아닌 통합의 업그레이드도 그 통합의 코어 명령은 새로 고치지만 확장이나 프리셋 레이어는 다시 등록하지 않습니다. 이후 그 통합에 `use`/`switch`를 실행해 다시 스캐폴딩하세요.

생성 파일 매니페스트가 없으면 업그레이드는 업그레이드할 것이 없다고 보고하고,
`--force`를 써도 설치된 어댑터 패키지, 생성된 파일, 로컬 복구 소유권을 그대로
둡니다. 대체 코드는 업그레이드가 관리 파일을 성공적으로 재생성한 뒤에만
영속화됩니다.

업그레이드가 통합에 프리셋 아티팩트가 등록된 상태에서 명령 레이아웃과 스킬 레이아웃 사이를 바꾸게 된다면, 파일을 변경하기 전에 업그레이드가 거부됩니다. 영향받는 프리셋을 제거하고, 레이아웃을 바꾸는 업그레이드를 실행한 다음, 프리셋을 다시 설치하세요.

외부 어댑터의 경우, 강제 업그레이드나 제거는 사용자 로컬 소유권 기록을 사용해
손상되었거나 로드할 수 없는 패키지를 복구할 수 있습니다. 그 증명이 없으면 이전
버전에만 있던 생성 파일은 수동 정리 경고와 함께 보존됩니다. 복구는 공유 이벤트
디스패처와 부분적으로 소유한 설정을 보존합니다. 신뢰된 대체 패키지는
`upgrade --force`에서 여전히 자신이 선언한 대상을 덮어쓸 수 있습니다.
복구는 카탈로그 정책이나 대체 패키지 신뢰를 우회하지 않습니다.

실패한 작업은 자신의 변경만 롤백하며, 관련 없는 사용자 파일이나 워크플로 진행
상황은 롤백하지 않습니다. 충돌하는 동시 편집은 복구 스냅샷을 유지한 채 보고됩니다.
어댑터 작성자는 안전한 쓰기와 복구를 위해
[설정 및 파일 소유권 계약](api.md#setup-and-file-ownership)과
[라이프사이클 보장](api.md#external-adapter-lifecycle-guarantees)을 따라야 합니다.

<a id="report-integration-status"></a>

## 통합 상태 보고

```bash
specify integration status
specify integration status --json
```

파일을 변경하지 않고 현재 프로젝트의 통합 상태를 보고합니다. 상태 보고서에는 기본
통합, 설치된 통합, 다중 설치 안전성, 누락된 관리 파일, 수정된 관리 파일, 유효하지
않은 매니페스트 경로, 공유 Spec Kit 인프라 상태, 확인되지 않은 매니페스트, 그리고
기본값에 민감한 공유 템플릿의 대상 통합이 포함됩니다. JSON 형식은 안정적인 기계
판독 가능 상태 데이터가 필요한 CI와 코딩 에이전트를 위한 것입니다. 상태 복구
휴리스틱이 기록된 파일과 다를 때는 기록된 원시 통합과 확인된 통합 매니페스트도
보고합니다.
보고서 상태가 `ok` 또는 `warning`이면 명령은 0으로 종료하고, `error`일 때만 1로
종료합니다. JSON 출력에서 `multi_install_safe`는 설치된 통합 집합을 평가할 수 없을
때 `null`입니다. 통합 상태가 없거나, 읽을 수 없거나, 유효한 기록된 통합 목록이
없거나, 설치된 통합이 기록되어 있지 않은 경우입니다.

<a id="catalog-management"></a>

## 카탈로그 관리

통합 카탈로그는 탐색 명령(`search`와 `info`)이 통합을 찾는 위치를 제어합니다. 카탈로그는 우선순위 순으로 확인됩니다.

카탈로그 관리, `integration list --catalog`, `integration info`는 어댑터 코드를
실행하지 않습니다. 일반적인 통합 목록 조회, 설정, 선택, 상태, 등록, 워크플로
디스패치는 신뢰된 설치 구현을 로드합니다.

### 카탈로그 나열

```bash
specify integration catalog list
```

활성 카탈로그 소스를 보여 줍니다. 프로젝트 수준 소스는(설정된 경우) 인덱스로 제거할 수 있으며, 그렇지 않으면 활성 소스는 제거 불가로 표시됩니다.

### 카탈로그 추가

```bash
specify integration catalog add <url>
```

| 옵션            | 설명                          |
| --------------- | ----------------------------- |
| `--name <name>` | 카탈로그의 선택적 이름        |

사용자 정의 카탈로그 URL을 프로젝트의 `.specify/integration-catalogs.yml`에
추가합니다. URL은 로컬 테스트를 포함해 HTTPS를 사용해야 합니다. 일반 HTTP는
localhost와 루프백 주소를 포함해 네트워크 접근 전에 거부됩니다.

같은 URL을 같은 이름으로 다시 추가하면 설정 변경 없이 성공합니다. 다른 이름은 거부됩니다.

### 카탈로그 제거

```bash
specify integration catalog remove <index>
```

`catalog list`의 0부터 시작하는 인덱스로 프로젝트 카탈로그 소스를 제거합니다.

### 카탈로그 해석 순서

처음으로 설정된 레이어가 하위 레이어를 대체합니다. 그 레이어 안에서는 숫자
우선순위가 낮은 쪽이 이깁니다.

1. **환경 변수** — `SPECKIT_INTEGRATION_CATALOG_URL`이 모든 카탈로그를 재정의합니다
2. **프로젝트 설정** — `.specify/integration-catalogs.yml`
3. **사용자 설정** — `~/.specify/integration-catalogs.yml`
4. **내장 기본값** — 공식 카탈로그 + 커뮤니티 카탈로그

<a id="historical-integration-metadata"></a>

### 과거 통합 메타데이터

통합 항목은 공개된 현재 릴리스에 대한 기존 최상위 필드를 유지하면서 과거 버전에서
메타데이터로의 `releases` 매핑을 추가할 수 있습니다. 단일 버전 항목은 `releases`
키 없이도 유효합니다. 예:

```json
{
  "id": "my-agent",
  "name": "My Agent",
  "version": "2.0.0",
  "description": "Current release",
  "releases": {
    "1.0.0": {
      "description": "Historical release",
      "repository": "https://example.com/my-agent/v1",
      "requires": {"speckit_version": ">=0.7"}
    }
  }
}
```

최상위 필드는 이전 클라이언트와 버전을 지정하지 않은 `search`, `info`,
`list --catalog`, `install` 명령에 대해 원래의 의미를 유지합니다. 과거 기록에는
`name`, `description`, `author`, `repository`, `license`, `tags`(문자열 목록),
`requires`(매핑)가 들어갈 수 있습니다. 공유되는 정체성, 이름, 작성자, 소스 정책은
유지됩니다. 설명, 저장소, 태그, 요구 사항 같은 현재 버전 전용 메타데이터는 해당
기록에 명시적으로 제공하지 않는 한 이전 릴리스에 상속되지 **않습니다**. 릴리스
기록은 `id`, `version`, `releases`, 소스 정책을 재정의할 수 없고, 다운로드 URL이나
아카이브를 공개할 수 없습니다.

프로그래밍 방식의 메타데이터 조회에서는
`IntegrationCatalog.get_integration_info(id, version="1.0.0")`이 이기는 소스에서
정확한 과거 기록을 선택하며, 그 버전이 없으면 `None`을 반환합니다.
`get_integration_versions(id)`는 같은 소스가 공개한 버전을 반환합니다. PEP 440상
동등한 표기(`v1.0`과 `1.0.0` 등)는 카탈로그 표기를 유지하면서 같은 릴리스와
일치합니다.
현재 릴리스는 `releases`에 반복되어서는 안 됩니다. 형식이 잘못된 기록과 중복된
동등 버전은 거부됩니다. 없는 버전이 더 낮은 우선순위의 소스로 넘어가거나 조용히
현재 버전을 선택하는 일은 없습니다. 이 조회들은 통합을 설치하지 않습니다.

## 통합별 옵션

에이전트별 설정 옵션은 `--integration-options`로 전달하세요. 의미, 예시,
마이그레이션 주의 사항은 해당 에이전트 가이드에 있습니다.

| 통합 | 가이드 |
| --- | --- |
| Generic | [출력 디렉터리와 스킬 모드](generic.md#output-options-and-invocation) |
| GitHub Copilot | [스킬 및 명령 레이아웃](copilot.md#commands-layout) |
| IBM Bob | [레거시 명령과 마이그레이션](bob.md#legacy-commands-and-migration) |
| Kimi Code | [레거시 마이그레이션](kimi.md#legacy-migration) |

그 밖의 에이전트별 세부 사항은 [통합 목록](index.md)에 나열되어 있습니다.

<a id="runtime-events"></a>

## 런타임 이벤트

이벤트를 지원하는 통합은 설정 `--integration-options` 문자열을 통해
`--events true|false`를 받습니다. 기본값은 활성화입니다. 핸들러 정의는 통합
기본값, 설치된 확장, `.specify/integration-events.yml`에서 옵니다. 이벤트를
활성화해도 설정된 핸들러가 없는 곳에 핸들러가 생기지는 않으며, 통합은 자신이
지원하는 네이티브 이벤트만 내보냅니다.

프로젝트 오버라이드를 설정하려면 `.specify/integration-events.yml`을 만드세요.
`<key>`를 통합 키로, 예시 명령을 설치되어 있고 스크립트로 뒷받침되는 Spec Kit
명령으로 바꾸세요.

```yaml
integrations:
  <key>:
    events:
      post_tool_use:
        - command: speckit.my-extension.check
          matcher: "*"
          timeout: 60
```

각 이벤트는 핸들러 매핑 또는 핸들러 목록을 받습니다. 핸들러에는 비어 있지 않은
`command` 문자열이 필요합니다. `matcher`는 선택적인 네이티브 필터 문자열이고,
`timeout`은 초 단위의 선택적인 양의 정수입니다.

통합 기본값과 확장 핸들러는 누적됩니다. 유효한 프로젝트 오버라이드는 오버라이드에
나열된 이벤트만이 아니라 **그 통합의 이벤트 맵 전체를 대체**합니다. 통합 키 아래에
`events: {}`를 설정하면 그 통합의 모든 핸들러가 비활성화됩니다. 오버라이드를
읽거나 파싱할 수 없거나 이벤트가 검증에 실패하면, Spec Kit은 경고하고 기본값과
확장 핸들러를 유지합니다. `--events false`는 이 모든 소스보다 우선합니다.

이벤트가 활성화된 상태에서는 `specify integration upgrade <key>`로 설정 변경을
적용하세요.

이벤트를 지원하는 통합에서 업그레이드 중에 Spec Kit 이벤트 핸들러를 비활성화하려면
다음과 같이 합니다.

```bash
specify integration upgrade <key> --integration-options="--events false"
```

`--integration-options`를 제공하면 저장된 설정 옵션이 대체되므로, 유지하고 싶은
다른 옵션을 반복해 적으세요. 이벤트 집합을 비활성화하면 새 훅 생성을 멈추는 데
그치지 않고 이전에 설치된 Spec Kit 훅이 제거됩니다.
에이전트별 가이드가 네이티브 이벤트 설정 경로와 제한 사항을 밝힙니다. 이 런타임
이벤트는 확장 명령 훅과는 별개입니다.

<a id="workflow-cli-dispatch"></a>

## 워크플로 CLI 디스패치

에이전트 가이드는 IDE 설정과 워크플로 CLI 지원을 구분합니다. 일부 IDE 통합은
별도의 CLI를 사용할 수 있으면 워크플로 스텝을 디스패치할 수 있고, 다른 통합은
명령/프롬프트 디스패치를 지원하지 않는다고 명시합니다. 어댑터 지원이란 Spec Kit이
호출을 구성할 수 있다는 뜻이지, 모든 벤더 CLI 릴리스가 그것을 상대로 검증되었다는
뜻이 아닙니다.

CLI를 쓸 수 있는 어댑터는 실행 파일 선택을 위한
`SPECKIT_INTEGRATION_<KEY>_EXECUTABLE`과 추가 CLI 인자를 위한
`SPECKIT_INTEGRATION_<KEY>_EXTRA_ARGS`를 지원합니다. 여기서 `<KEY>`는 대문자이며
하이픈은 밑줄로 바뀝니다. 추가 인자는 POSIX 셸 인용 규칙으로 파싱됩니다. 배치
위치와 중복 플래그 동작은 어댑터마다 다릅니다. 특히 Docker Agent와 Goose는
에이전트 가이드를 참고하세요.

설정 `--integration-options`는 워크플로의
[`integration_args`와 `integration_options`](../workflow-steps/types.md#per-step-integration-configuration)와
별개입니다. 어댑터는 구현하지 않는 한 비어 있지 않은 스텝별 설정을 거부합니다.
Docker Agent는 현재 구조화된 에이전트/안전 설정을 노출합니다.
헤드리스 승인 플래그, 모델 전달, 구조화된 출력 지원도 어댑터별이며 보편적인
보장이 아닙니다.

반면 `command` 및 `prompt` 스텝의
[`reasoning_effort`와 `context_size`](../workflow-steps/types.md#reasoning-effort-and-context-size)
설정은 하나의 공통 계약을 따릅니다. Spec Kit은 각 값을 확인하고, 그 설정을
선언하고 변환하는 통합에만 전달합니다. 다른 통합에서는 CLI가 실행되기 전에 스텝이
실패합니다. GitHub Copilot과 Codex CLI는 두 설정을 모두 변환하고, Claude Code는
`reasoning_effort`를 변환합니다. 에이전트 가이드가 각 매핑을 설명합니다.
어댑터 작성자는 [통합 API 레퍼런스](api.md#per-step-dispatch-settings)에 설명된
대로 참여를 선언합니다.

## FAQ

<a id="can-i-install-multiple-integrations-in-the-same-project"></a>

### 같은 프로젝트에 여러 통합을 설치할 수 있나요?

네, 하지만 기본 구성보다는 팀 이식성을 위한 것입니다. 여러 통합은 설치된 통합과 새 통합이 모두 Spec Kit에 의해 다중 설치 안전으로 선언된 경우에만 자동으로 허용됩니다. 그 밖의 조합에서는 `--force`를 전달해 여러 에이전트가 관련 없는 에이전트별 지침이나 명령을 보게 될 수 있음을 인정하세요.

Spec Kit은 `.specify/integration.json`에서 `default_integration`으로 기본 통합 하나를, `installed_integrations`로 설치된 모든 통합을, `integration_settings`로 통합별 런타임 설정을, 그리고 향후 상태 마이그레이션을 위한 전용 `integration_state_schema`를 추적합니다. 레거시 `integration` 필드는 기본 통합의 별칭으로 남아 있습니다.

### 어떤 통합이 다중 설치 안전인가요?

에이전트 파일을 생성하는 통합의 경우, 안전하려면 정적이고 고유한 출력 경로, 안정적인
명령 호출 설정, 그리고 관리 파일이 다른 안전한 통합과 겹치지 않는 별도의
매니페스트가 필요합니다. 레지스트리 테스트가 이 불변 조건을 강제합니다. 공유 Spec
Kit 템플릿은 단일 기본값에 맞춰진 상태로 유지됩니다.

각 통합의 선언은 `specify integration list`로 확인하세요.
각 통합의 가이드가 안전성 상태와 설치되는 명령 디렉터리를 문서화합니다.

다른 통합과 명령 디렉터리를 공유하거나, `--commands-dir` 같은 동적 설치 경로가 필요하거나, 공유 도구 설정을 병합하는 통합은 기본적으로 안전하다고 선언되지 않습니다. 그래도 `--force`를 쓰면 다른 통합과 나란히 설치할 수 있습니다.

### 제거하거나 전환하면 제 변경 사항은 어떻게 되나요?

수정한 매니페스트 추적 파일은 자동으로 보존됩니다. 수정되지 않은 추적 파일(원래의
SHA-256 해시와 일치)만 제거됩니다. 이를 재정의하려면 `--force`를 사용하세요.
통합별 정리 예외는 [제거](#uninstall-an-integration)를 참고하세요.

### 어떤 키를 써야 하는지 어떻게 아나요?

`specify integration list`를 실행해 사용 가능한 모든 통합과 키를 확인하거나,
[통합 목록](index.md#choose-an-integration)을 확인하세요.

### 통합을 사용하려면 AI 코딩 에이전트가 설치되어 있어야 하나요?

CLI 기반 통합은 도구가 설치되어 있어야 합니다. IDE 기반 통합은 IDE 자체를 통해
동작하며 설정에 별도의 CLI가 필요하지 않을 수 있습니다. 워크플로 디스패치에는
추가 CLI 요구 사항이 있을 수 있습니다. 사용하는 통합의 가이드를 참고하세요.
`specify integration list`는 각 통합의 설정 요구 사항을 보여 줍니다.

통합에 명령 템플릿이 없으면, `specify init`은 사용할 수 없는 에이전트 명령을
권장하는 대신 프로젝트 디렉터리 안내를 유지하고 `specify integration list`로
통합을 고르고 `specify integration switch <key>`로 전환하는 방법을 보여 줍니다.

### `upgrade`와 `switch`는 각각 언제 써야 하나요?

Spec Kit을 업그레이드했고 설치된 통합의 관리 파일을 새로 고치고 싶을 때는 `upgrade`를 사용하세요. 현재 기본값을 다른 통합으로 교체하고 싶을 때는 `switch`를 사용하세요. 대상이 이미 설치되어 있으면 `switch`는 `use`처럼 동작합니다.

### 제가 설치한 확장과 프리셋은 설치된 모든 통합에 적용되나요?

아니요. 확장(`specify extension add`)과 프리셋(`specify preset add`)은 다른 통합이 설치되어 있더라도 현재 활성(기본) 통합에만 명령 오버라이드를 등록합니다. 기본이 아닌 통합은 기본값이 될 때까지 그 아티팩트를 받지 않습니다. `specify integration use <key>`(또는 `switch <key>`)가 새로 활성화된 통합에 활성화된 모든 확장과 프리셋을 다시 스캐폴딩합니다. `specify integration upgrade`도 같은 규칙을 따릅니다. 활성 통합을 업그레이드할 때만 확장과 프리셋을 다시 등록합니다.
