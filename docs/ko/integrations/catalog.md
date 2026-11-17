# Spec Kit 통합 카탈로그

통합 카탈로그는 Spec Kit용 AI 에이전트 통합의 탐색, 버전 관리, 배포를 가능하게 합니다.

## 카탈로그 파일

### 내장 카탈로그 (`integrations/catalog.json`)

Spec Kit과 함께 제공되는 통합을 담습니다. 코어 팀이 유지 관리하며 항상 설치할 수 있습니다.

### 커뮤니티 카탈로그 (`integrations/catalog.community.json`)

커뮤니티가 기여한 통합입니다. 기본 커뮤니티 소스는 탐색 전용입니다. 어댑터가
등재되었다는 것은 설치 허가도, 코드 감사도 아닙니다. 설치가 활성화된 카탈로그를
사용하기 전에 외부 코드를 검토하세요.

## 카탈로그 설정

처음으로 설정된 레이어가 하위 레이어를 대체합니다. 그 레이어 안에서는 숫자
우선순위가 낮은 쪽이 이깁니다.

1. **환경 변수** — `SPECKIT_INTEGRATION_CATALOG_URL`이 단일 URL로 모든 카탈로그를 재정의합니다
2. **프로젝트 설정** — 프로젝트 루트의 `.specify/integration-catalogs.yml`
3. **사용자 설정** — 사용자 홈 디렉터리의 `~/.specify/integration-catalogs.yml`
4. **내장 기본값** — `catalog.json` + `catalog.community.json`

첫 번째 프로젝트 카탈로그를 추가하면, 기본 소스도 프로젝트 파일에 나열하지 않는 한
기본 소스가 대체됩니다. 공통 탐색, 신뢰, 비공개 소스 안내는
[카탈로그](../overview/catalogs.md)를 참고하세요.

`integration-catalogs.yml` 예시:

```yaml
catalogs:
  - url: "https://example.com/my-catalog.json"
    name: "my-catalog"
    priority: 1
    install_allowed: true
```

## CLI 명령

`specify integration list --catalog`, `search`, `info`로 둘러보세요.
[카탈로그 관리](cli.md#catalog-management)와
[외부 어댑터 설치](cli.md#catalog-installed-external-adapters)는 CLI 레퍼런스를
참고하세요.

<a id="integration-descriptor-integrationyml"></a>

## 통합 디스크립터 (`integration.yml`)

각 외부 통합 패키지에는 어댑터 전용 `integration.yml` 디스크립터와
`IntegrationBase` 하위 클래스를 내보내는 루트 `__init__.py`가 포함됩니다.
명령 목록이나 복사된 코어 템플릿은 필요하지 않습니다.

```yaml
schema_version: "1.0"
integration:
  id: "sample-agent"
  name: "Sample Agent"
  version: "1.0.0"
  description: "Adapter for Sample Agent"
  license: "MIT"
requires:
  speckit_version: ">=1.1.2.dev0"
  tools:
    - name: "sample-agent"
      required: true
```

필수 디스크립터 필드:

| 필드 | 검증 |
|-------|------------|
| `schema_version` | 문자열 `"1.0"`이어야 합니다 |
| `integration.id` | 외부 ID는 `^[a-z0-9][a-z0-9-]*$`와 일치해야 합니다. `con`, `prn`, `aux`, `nul`, `com1`~`com9`, `lpt1`~`lpt9`는 예약된 이름입니다 |
| `integration.name`, `integration.description` | 문자열이어야 합니다 |
| `integration.version` | PEP 440 버전 문자열이어야 합니다 |
| `requires.speckit_version` | 필수이며 비어 있지 않은 PEP 440 버전 지정자 문자열 |

`requires.tools`는 선택 사항입니다. 필요한 실행 파일이 없는 어댑터에서는
생략하세요. 선택적인 레거시 `provides` 메타데이터는 여전히 유효하지만 호스트
명령을 제공하지 않습니다. 클래스 메타데이터, 런타임 메서드, 도구, 스토리지 요구
사항은 [통합 API 레퍼런스](api.md)를 참고하세요.

<a id="catalog-schema"></a>

## 카탈로그 스키마

두 카탈로그 파일은 같은 JSON 스키마를 따릅니다.

```json
{
  "schema_version": "1.0",
  "updated_at": "2026-04-08T00:00:00Z",
  "catalog_url": "https://...",
  "integrations": {
    "sample-agent": {
      "id": "sample-agent",
      "name": "Sample Agent",
      "version": "1.0.0",
      "description": "Adapter for Sample Agent",
      "download_url": "https://example.com/sample-agent/1.0.0/sample-agent.zip",
      "tags": ["cli"]
    }
  }
}
```

### 최상위 필드

| 필드 | 타입 | 설명 |
|-------|------|-------------|
| `schema_version` | string | `"1.0"`이어야 합니다 |
| `updated_at` | string | 선택적 ISO 8601 타임스탬프 |
| `integrations` | object | 통합 ID → 메타데이터 맵 |

### 통합 항목 필드

| 필드 | 타입 | 필수 | 설명 |
|-------|------|----------|-------------|
| `id` | string | 아니요 | 선택적인 명시적 ID. 맵 키와 일치해야 합니다 |
| `name` | string | 예 | 사람이 읽을 수 있는 표시 이름 |
| `version` | string | 예 | PEP 440 버전(예: `1.0.0`, `1.0.0a1`) |
| `description` | string | 예 | 한 줄 설명 |
| `author` | string | 아니요 | 작성자 이름 또는 조직 |
| `repository` | string | 아니요 | 소스 저장소 URL |
| `license` | string | 아니요 | 디스크립터와 일치하는 라이선스 식별자 |
| `tags` | array | 아니요 | 검색 가능한 태그(예: `["cli", "ide"]`) |
| `download_url` | string | 외부 설치 시 | 고정된 ZIP, tar.gz, tgz 아카이브 URL. HTTPS 또는 루프백 HTTP |
| `sha256` | string | 아니요 | 아카이브의 64자 16진수 SHA-256 |
| `requires` | object | 아니요 | 제공하는 경우 디스크립터 요구 사항과 일치해야 합니다 |
| `releases` | object | 아니요 | [과거 버전 메타데이터](cli.md#historical-integration-metadata). 설치 가능한 릴리스가 아닙니다 |

맵 키와 선언된 `id`는 `integration.id`와 일치해야 합니다. 이름, 버전, 설명, 그리고
선택적인 작성자/저장소/라이선스 메타데이터는 디스크립터와 일치해야 합니다. 내장
항목은 구현이 CLI와 함께 제공되므로 다운로드 필드가 필요 없습니다.

## 설치 정책과 스토리지

`integration catalog add`로 카탈로그를 등록하면 설치가 활성화된 프로젝트 소스가
만들어집니다. 탐색만 허용하려면 설정에서 `install_allowed: false`를 지정하세요.
이 정책은 `--trust-integration`이나 `--force`로 재정의할 수 없습니다. 각 외부
설치/업데이트는 `--trust-integration`으로 명시적으로 사전 승인하지 않는 한
Python을 다운로드/임포트하기 전에 묻습니다. 인증이 필요한 GitHub 자산은 기존
Spec Kit 인증 제공자를 사용합니다.

설치된 코드는 `.specify/integrations/packages/<id>/`에 저장되고 출처와 해시는
`packages.json`에 기록됩니다. 둘 다 관리형 `.specify/.gitignore`에 의해
제외됩니다. 실행 동의는 프로젝트, 통합 ID, 검증된 패키지 다이제스트에 묶여
`~/.specify/integration-trust.json`에 별도로 저장됩니다. 생성된 파일에는 해시로
추적되는 자체 `<id>.manifest.json`이 있습니다.

설치된 패키지 코드를 제자리에서 편집하지 마세요. 새 아카이브/버전을 게시하고
업그레이드하세요. [신뢰와 재승인](cli.md#catalog-installed-external-adapters),
[업그레이드와 복구](cli.md#upgrade-an-integration)는 CLI 레퍼런스를 참고하세요.

## 기여하기

내장 통합을 추가하거나 외부 통합 카탈로그 항목을 제출하려면
[기여 가이드](contributing.md)를 참고하세요. 에이전트별 설정은
[통합 목록](index.md)에서 시작하세요. 공통 사용자용 명령은
[통합 관리 레퍼런스](cli.md)에 문서화되어 있습니다.

## 함께 보기

- [통합 API 레퍼런스](api.md)
- [커뮤니티 통합](../community/integrations.md)
