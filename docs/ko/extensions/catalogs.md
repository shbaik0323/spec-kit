# 확장 카탈로그

확장 카탈로그는 `specify extension search`와 `add`가 확장을 찾는 위치를
제어합니다. 카탈로그 스택은 함께 검색되는 소스의 순서 있는 목록입니다.
같은 확장 ID가 둘 이상의 소스에 나타나면 카탈로그 우선순위 숫자가 낮은 쪽이
우선합니다.

<a id="trust-model-discovery-only-vs-install-sources"></a>

## 신뢰 모델: 탐색 전용 소스와 설치 소스

카탈로그에는 두 종류가 있으며, 그 구분은 **보안 경계**입니다.

- **설치 소스**(`install_allowed: true`)는 설치 출처로 신뢰하는 카탈로그입니다.
  내장 `default` 카탈로그가 그중 하나이고, 여러분이 직접 작성하고 검증한
  카탈로그도 그렇습니다.
- **탐색 전용** 카탈로그(`install_allowed: false`)는 확장을 찾기 위한 검색 가능한
  표면이지만 설치할 수는 없습니다. 내장 `community` 카탈로그는 이미 검색에
  활성화되어 있으므로 추가할 필요가 없습니다.

| 우선순위 | 내장 카탈로그 | 설치 허용 | 용도 |
| --- | --- | --- | --- |
| 1 | `catalog.json` (`default`) | 예 | 설치 가능한 확장 |
| 2 | `catalog.community.json` (`community`) | 아니요 | 커뮤니티 확장 탐색 |

커뮤니티 항목은 검증되지 않은 서드파티 코드입니다. 메인테이너는 확장 코드가 아니라
카탈로그 메타데이터를 확인합니다. 확장은 설치하기 전에 직접 검증해야 합니다.

> [!WARNING]
> 검증되지 않은 탐색 전용 카탈로그를 `install_allowed`로 바꾸지 마세요. 탐색과
> 설치 사이의 분리가 무력화됩니다.

커뮤니티 카탈로그에서 찾은 것을 설치하는 방법은 두 가지입니다.

1. **검증한 단일 릴리스를 직접 설치합니다.** `specify extension info <name>`에서
   후보 아카이브 URL을 얻으세요. 설치되지 않은 탐색 전용 항목에 대해 "Candidate
   archive" URL을 출력합니다. 그 릴리스 아카이브를 검토하기 전까지는 URL을 신뢰할
   수 없는 것으로 취급하고, 검토한 뒤 설치하세요.

   ```bash
   specify extension info <name>
   specify extension add <name> --from <archive-url>
   ```

2. **자신의 카탈로그를 선별합니다.** 통제되고 재사용 가능한 조직 소스를 원한다면
   릴리스를 직접 검토하고, 그 통제된 카탈로그만 `install_allowed: true`로
   표시하세요.

## 카탈로그 해석 순서

활성 스택은 다음 순서로 선택됩니다.

1. **환경 변수:** `SPECKIT_CATALOG_URL`이 스택 전체를 재정의합니다.
2. **프로젝트 설정:** `.specify/extension-catalogs.yml`.
3. **사용자 설정:** `~/.specify/extension-catalogs.yml`.
4. **내장 기본값:** `default`와 `community`.

기존 프로젝트 설정은 사용자 설정과 내장 기본값보다 우선합니다. `catalogs: []`를
포함해 비어 있거나 형식이 잘못된 파일은 다른 소스로 폴백하는 대신 검증 오류와 함께
차단됩니다(fail closed).
해석 순서의 다음 소스를 사용하려면 설정 파일을 제거하거나, 유효한 카탈로그 항목으로
채우세요.
선택된 스택 안에서 카탈로그는 우선순위로 확인되며, 처음으로 일치하는 확장 ID가
이깁니다. 정확한 버전 조회는 이기는 항목에 요청한 버전이 없을 때 다른 카탈로그로
폴백하지 않습니다.

## 카탈로그 관리

```bash
specify extension catalog list
specify extension catalog --help

# 직접 소유하고 검증한 내부 소스 추가
specify extension catalog add \
  --name "internal" \
  --priority 2 \
  --install-allowed \
  --description "Our approved extensions" \
  https://internal.company.com/spec-kit/catalog.json

# 탐색 전용 소스 추가(기본 권한)
specify extension catalog add \
  --name "partner" \
  --priority 5 \
  https://partner.example.com/spec-kit/catalog.json

specify extension catalog remove internal
```

이 명령들은 프로젝트의 `.specify/extension-catalogs.yml`을 편집합니다.
같은 이름의 카탈로그를 동일한 설정으로 다시 추가하면 설정 변경 없이 성공합니다.
다른 설정은 거부됩니다. 전체 옵션 레퍼런스는
[카탈로그 관리](cli.md#catalog-management)에 있습니다.

설정을 직접 편집할 수도 있습니다.

```yaml
catalogs:
  - name: "default"
    url: "https://raw.githubusercontent.com/github/spec-kit/main/extensions/catalog.json"
    priority: 1
    install_allowed: true
    description: "Built-in catalog of installable extensions"

  - name: "internal"
    url: "https://internal.company.com/spec-kit/catalog.json"
    priority: 2
    install_allowed: true
    description: "Our approved extensions"

  - name: "community"
    url: "https://raw.githubusercontent.com/github/spec-kit/main/extensions/catalog.community.json"
    priority: 3
    install_allowed: false
    description: "Community-contributed extensions (discovery only)"
```

## 조직 카탈로그 사용자 정의

조직은 사용 가능한 릴리스를 통제하거나, 내부 도구를 연동하거나, 컴플라이언스 요구
사항을 충족하거나, 에어갭 네트워크를 지원하기 위해 비공개 카탈로그를 사용합니다.

### 카탈로그 만들기

팀이 검증한 릴리스를 담은 `catalog.json` 파일을 만드세요.

```json
{
  "schema_version": "1.0",
  "updated_at": "2026-02-03T00:00:00Z",
  "catalog_url": "https://your-org.com/spec-kit/catalog.json",
  "extensions": {
    "jira": {
      "name": "Jira Integration",
      "id": "jira",
      "description": "Create Jira issues from spec-kit artifacts",
      "author": "Your Organization",
      "version": "2.1.0",
      "download_url": "https://github.com/your-org/spec-kit-jira/archive/refs/tags/v2.1.0.zip",
      "repository": "https://github.com/your-org/spec-kit-jira",
      "license": "MIT",
      "requires": {
        "speckit_version": ">=0.1.0",
        "tools": [
          {"name": "atlassian-mcp-server", "required": true}
        ]
      },
      "provides": {
        "commands": 3,
        "hooks": 1
      },
      "tags": ["jira", "atlassian", "issue-tracking"],
      "verified": true
    },
    "internal-tool": {
      "name": "Internal Tool Integration",
      "id": "internal-tool",
      "description": "Connect to internal company systems",
      "author": "Your Organization",
      "version": "1.0.0",
      "download_url": "https://internal.your-org.com/extensions/internal-tool-1.0.0.zip",
      "repository": "https://github.internal.your-org.com/spec-kit-internal",
      "license": "Proprietary",
      "requires": {
        "speckit_version": ">=0.1.0"
      },
      "provides": {
        "commands": 2
      },
      "tags": ["internal", "proprietary"],
      "verified": true
    }
  }
}
```

### 카탈로그 호스팅 및 설정

| 방법 | URL 예시 | 사용 사례 |
| --- | --- | --- |
| GitHub Pages | `https://your-org.github.io/spec-kit-catalog/catalog.json` | 공개 또는 조직 내 공개 |
| 내부 웹 서버 | `https://internal.company.com/spec-kit/catalog.json` | 기업 네트워크 |
| S3/클라우드 스토리지 | `https://s3.amazonaws.com/your-bucket/catalog.json` | 클라우드에 호스팅하는 팀 |
| 로컬 HTTPS 서버 | `https://localhost:8443/catalog.json` | Python이 신뢰하는 인증서를 쓰는 개발/테스트 |

카탈로그 URL은 localhost를 포함해 HTTPS를 사용해야 합니다. 일반 HTTP 카탈로그는
네트워크 접근 전에 거부됩니다.
`specify extension catalog add`로 카탈로그를 추가하거나 다음을 사용하세요.

```yaml
# .specify/extension-catalogs.yml
catalogs:
  - name: "my-org"
    url: "https://your-org.com/spec-kit/catalog.json"
    priority: 1
    install_allowed: true
    description: "Our approved extensions"
```

CI/CD나 단일 카탈로그 환경에서는 스택 전체를 교체하세요.

```bash
export SPECKIT_CATALOG_URL="https://your-org.com/spec-kit/catalog.json"
```

결과를 확인합니다.

```bash
specify extension catalog list
specify extension search
specify extension add jira
```

### GitHub에 호스팅된 비공개 카탈로그

`GH_TOKEN` 또는 `GITHUB_TOKEN`이 GitHub 호스팅 URL(`raw.githubusercontent.com`,
`github.com`, `api.github.com`, `codeload.github.com`)에 대한 요청을 인증합니다.
비공개 카탈로그 JSON과 확장 ZIP에는 관련 저장소에 대한 접근 권한이 필요합니다.
GitHub가 아닌 카탈로그 URL은 이 자격 증명 없이 가져옵니다.

```bash
export GITHUB_TOKEN=$(gh auth token)
specify extension search jira
specify extension add jira-sync
```

### 선별된 환경과 에어갭 환경

카탈로그는 `jira`와 `github`처럼 승인된 확장만 나열할 수 있습니다.
그 항목만 검색에 나타나야 한다면 그것을 유일한 카탈로그로 설정하세요.
비공개/독점 확장은 내부 아카이브 호스트를 가리킬 수 있습니다. 공개 카탈로그 제출은
선택 사항입니다.

에어갭 네트워크의 경우, 검증한 확장 ZIP을 내부 파일 서버로 다운로드하고, 그 내부
URL을 가리키는 카탈로그를 만들고, 네트워크 안에 카탈로그를 호스팅하세요. 예를 들어
항목은 `"download_url": "https://files.internal/spec-kit/jira-2.1.0.zip"`을 사용할
수 있습니다.

### 개발과 테스트

`./my-catalog/`를 브라우저나 운영 체제만이 아니라 Python의 TLS 클라이언트가
신뢰하는 인증서를 가진 HTTPS 서버로 제공하세요. 개발용 CA라면 CLI를 실행하기 전에
`SSL_CERT_FILE`이 그 CA를 포함한 PEM CA 번들을 가리키게
하세요([TLS 인증서](../reference/authentication.md#tls-certificates) 참고).

```bash
export SSL_CERT_FILE="/absolute/path/to/ca-bundle.pem"
export SPECKIT_CATALOG_URL="https://localhost:8443/catalog.json"
specify extension add my-new-extension
```

카탈로그 설정이 직접 설치를 막지는 않습니다.

```bash
specify extension add jira
specify extension add <extension-name> --from https://github.com/someone/spec-kit-ext/archive/refs/tags/v1.0.0.zip
specify extension add --dev /path/to/extension
```

직접 URL 설치는 아카이브가 설정된 카탈로그를 통해 선택된 것이 아니므로 보안 경고를
표시합니다. 설치하기 전에 검증하세요.

<a id="catalog-entry-schema"></a>

## 카탈로그 항목 스키마

다음은 카탈로그 필드를 설명합니다. 커뮤니티 제출 요구 사항은 완전한 설명
메타데이터도 요구합니다.

카탈로그 루트에는 `schema_version`과 `extensions` 매핑이 있어야 합니다.
각 매핑 키가 권위 있는 확장 ID입니다. 항목 안에서 반복되는 `id` 필드는 선택
사항입니다.

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| `name` | string | 예 | 사람이 읽을 수 있는 이름 |
| `id` | string | 아니요 | 권위 있는 매핑 키의 선택적 반복 |
| `version` | string | 예 | 현재 확장 버전 |
| `download_url` | string | 아카이브 릴리스의 경우 | 릴리스 아카이브의 URL. 번들 항목은 생략 가능 |
| `bundled` | boolean | 아니요 | Spec Kit과 함께 제공되는 확장임을 표시. 실행 중인 배포판에서 해석됨 |
| `repository` | string | 아니요 | 소스 코드 URL(커뮤니티 제출에는 필수) |
| `description` | string | 예 | 간단한 설명. 커뮤니티 제출은 200자 미만을 요구 |
| `author` | string | 아니요 | 작성자/조직(제출에는 필수) |
| `license` | string | 아니요 | SPDX 라이선스 식별자(제출에는 필수) |
| `homepage`, `documentation`, `changelog` | string | 아니요 | 관련 URL |
| `sha256` | string | 아니요 | 아카이브 SHA-256 다이제스트. 제공되면 설치 전에 검증됨 |
| `requires.speckit_version` | string | 아니요 | 버전 제약(제출에는 필수) |
| `requires.tools` | array | 아니요 | 필수인 `name`, 선택적인 `version`, `required`(기본값 `false`)를 가진 객체 |
| `provides.commands`, `provides.hooks` | number | 아니요 | 기능 수 |
| `tags` | array | 아니요 | 검색 태그(2~10개 권장) |
| `verified` | boolean | 아니요 | 메인테이너가 설정하는 메타데이터 플래그. 기본값 `false`. 코드 감사 보증이 아님 |
| `created_at`, `updated_at` | string | 아니요 | ISO 8601 타임스탬프 |
| `releases` | object | 아니요 | 버전을 키로 하는 과거 릴리스 |

<a id="historical-releases"></a>

### 과거 릴리스

카탈로그는 현재 릴리스를 최상위 필드에 유지하고 이전 릴리스를 `releases` 매핑에
추가할 수 있습니다. 오래된 단일 버전 카탈로그는 변경 없이 계속 동작합니다. 각 과거
릴리스에는 자체 다운로드 URL과 SHA-256 다이제스트가 필요합니다. 릴리스별 요구
사항이나 제공 기능은 현재 릴리스에서 상속되지 않으므로 그 릴리스의 기록에 넣어야
합니다. 현재 릴리스와 마찬가지로 다이제스트에는 대소문자를 구분하지 않는 `sha256:`
접두사와 앞뒤 공백을 쓸 수 있습니다.

```json
{
  "extensions": {
    "my-extension": {
      "name": "My Extension",
      "version": "0.5.1",
      "download_url": "https://example.com/my-extension-0.5.1.zip",
      "sha256": "<64-character SHA-256 for 0.5.1>",
      "releases": {
        "0.4.12": {
          "download_url": "https://example.com/my-extension-0.4.12.zip",
          "sha256": "<64-character SHA-256 for 0.4.12>"
        }
      }
    }
  }
}
```

예시는 간결함을 위해 다른 메타데이터를 생략합니다. 현재 버전은 `releases`에
반복되어서는 안 됩니다. 형식이 잘못되었거나 중복된 릴리스 기록은 거부됩니다.
`v1.0`과 `1.0` 같은 동등한 PEP 440 표기는 같은 릴리스를 선택합니다. 카탈로그가
공개한 표기는 그대로 보입니다.

카탈로그 기반 번들의 확장 고정도 이기는 소스에서 과거 릴리스를 선택하며, 그 릴리스
자체의 아카이브와 다이제스트를 사용하고 다운로드한 매니페스트의 ID와 버전을
확인합니다. 번들된 확장은 패키지 버전 정책을 따릅니다. 더 새로운 호환 패키지
패치가 경고와 함께 사용될 수 있습니다.
번들 설치는 [번들 CLI 레퍼런스](../bundles/cli.md#install-a-bundle)를 참고하세요.

## 함께 보기

- [확장 사용자 가이드](overview.md)
- [확장 CLI 레퍼런스](cli.md)
- [확장 게시하기](publishing.md)
- [커뮤니티 확장](../community/extensions.md)
- [공통 카탈로그 레퍼런스](../overview/catalogs.md)
