# 프리셋 카탈로그

카탈로그는 프리셋을 탐색하고 ID 기반 설치를 위한 릴리스를 선택합니다.
카탈로그의 우선순위는 카탈로그 소스의 순서를 정하며, 아티팩트 해석을 결정하는
설치된 프리셋의 우선순위와는 독립적입니다.

## 탐색과 신뢰

내장 스택에는 다음이 있습니다.

| 이름 | 소스 | 기본 권한 |
| --- | --- | --- |
| `default` | [퍼스트파티 프리셋 카탈로그](https://github.com/github/spec-kit/blob/main/presets/catalog.json) | 설치 허용 |
| `community` | [커뮤니티 프리셋 카탈로그](https://github.com/github/spec-kit/blob/main/presets/catalog.community.json) | 탐색 전용 |

커뮤니티 프리셋은 독립적으로 유지 관리됩니다. 메인테이너는 카탈로그 항목의
완전성과 형식을 확인할 뿐, 프리셋 코드의 안전성이나 품질을 확인하지 않습니다. 그
코드를 검토, 감사, 보증, 지원하지 않습니다.
사용자는 설치하고 사용하기 전에 프리셋의 소스를 검증해야 합니다.

> [!IMPORTANT]
> 프로젝트의 `.specify/preset-catalogs.yml`은 `add`와 `search`가 여러분이 선택하지
> 않은 카탈로그를 가리키게 할 수 있습니다. 낯선 프로젝트에서는
> `specify preset catalog list`를 실행하고 설치 허용 소스를 확인하세요. 프로젝트
> 설정과 `install_allowed: true`는 코드 검토의 증거가 아닙니다. 직접 소유하거나
> 검증한 카탈로그로부터의 설치만 허용하고, 낯선 소스와 커뮤니티 소스는 탐색
> 전용으로 두세요.

커뮤니티 릴리스를 검증한 뒤에는 명시적인
`specify preset add --from <archive-url>`이 카탈로그 조회 없이 그 아카이브를
설치합니다. 이는 소스 선택이지, 검토되지 않은 콘텐츠를 안전하게 만드는 우회로가
아닙니다.

## 카탈로그 설정

처음으로 사용 가능한 설정 수준이 스택 전체를 제공합니다. 프로젝트 및 사용자 항목이
기본값을 자동으로 보강하지 않습니다.

| 순서 | 소스 |
| --- | --- |
| 첫째 | `SPECKIT_PRESET_CATALOG_URL` 환경 변수 |
| 다음 | 프로젝트 `.specify/preset-catalogs.yml` |
| 다음 | 사용자 `~/.specify/preset-catalogs.yml` |
| 마지막 | 내장 `default` 및 `community` 카탈로그 |

환경 변수는 설치가 허용된 `custom`이라는 이름의 카탈로그 하나를 선택합니다. 검증한
소스에만 사용하세요. 유효한 카탈로그 항목이 없는 설정 파일은 다음 수준으로
넘어갑니다. 형식이 잘못된 YAML이나 유효하지 않은 설정 URL/우선순위는 다른 소스를
조용히 선택하는 대신 검증 오류를 발생시킵니다.

프로젝트 설정 예시:

```yaml
catalogs:
  - name: "my-org-presets"
    url: "https://catalog.example.com/presets.json"
    priority: 5
    install_allowed: true
    description: "Our vetted presets"
```

| 항목 필드 | 의미 |
| --- | --- |
| `url` | 로컬 개발을 포함해 HTTPS 카탈로그 URL. 일반 HTTP는 거부됨 |
| `name` | CLI가 표시하고 출처 기록에 사용하는 소스 이름 |
| `priority` | 숫자가 낮을수록 먼저 확인됨 |
| `install_allowed` | ID 기반 설치를 허용하는지 여부. 기본값 `false` |
| `description` | 선택적인 소스 설명 |

프로젝트 항목은 CLI로 관리하세요.

```bash
specify preset catalog list
specify preset catalog add https://catalog.example.com/presets.json \
  --name my-org-presets --priority 5 --install-allowed
specify preset catalog remove my-org-presets
```

`--install-allowed`는 소스를 검증한 뒤에만 사용하세요. 생략하면 탐색 전용
카탈로그가 추가됩니다. 첫 번째 프로젝트 항목을 추가하면 암묵적인 내장 스택이
대체됩니다. 다른 소스가 여전히 필요하다면 명시적으로 포함하세요.
사용자 수준 파일은 `catalog add`가 아니라 별도로 설정합니다.

## 항목 스키마

카탈로그 JSON에는 `schema_version` 필드와 프리셋 ID를 키로 하는 `presets` 객체가
있습니다. 다음은 비공개 카탈로그의 다운로드 가능한 현재 릴리스를 보여 줍니다.
사용하기 전에 예시 URL과 다이제스트를 교체하세요.

```json
{
  "schema_version": "1.0",
  "updated_at": "2026-11-03T00:00:00Z",
  "presets": {
    "team-traceability": {
      "id": "team-traceability",
      "name": "Team Traceability",
      "description": "Adds team traceability requirements to specifications",
      "author": "Example Team",
      "version": "2.0.0",
      "download_url": "https://github.com/example/team-traceability/archive/refs/tags/v2.0.0.zip",
      "sha256": "<64 hexadecimal digits for this archive>",
      "repository": "https://github.com/example/team-traceability",
      "documentation": "https://github.com/example/team-traceability/blob/main/README.md",
      "license": "MIT",
      "requires": {
        "speckit_version": ">=0.9.0"
      },
      "provides": {
        "templates": 1,
        "commands": 0,
        "scripts": 0
      },
      "tags": ["traceability", "governance"],
      "created_at": "2026-11-03",
      "updated_at": "2026-11-03"
    }
  }
}
```

카탈로그의 `provides`는 매니페스트의 `provides.templates` 목록과 달리 기여 수를
담습니다. 메타데이터를 게시된 매니페스트와 일관되게 유지하세요. `bundled: true`는
Spec Kit 안에 포함되어 제공되는 퍼스트파티 프리셋을 기술할 수 있습니다. 그런
항목은 다운로드 URL이 없을 수 있으며 네트워크 다운로드가 아니라 일치하는 패키지
파일을 필요로 합니다.

단일 버전 레거시 항목은 여전히 유효합니다. 현재 릴리스는 `sha256`을 생략할 수
있습니다. 있으면 설치 전에 다이제스트가 확인됩니다.
커뮤니티 제출 자동화는 제출자가 제공하지 않았더라도 다운로드한 아카이브의
다이제스트를 계산해 기록합니다.

## 과거 릴리스

이전 릴리스를 공개하려면 버전을 키로 하는 `releases` 매핑을 추가하세요.

```json
{
  "version": "2.0.0",
  "download_url": "https://example.com/team-traceability-2.0.0.zip",
  "sha256": "<64 hexadecimal digits for the current archive>",
  "releases": {
    "1.5.0": {
      "download_url": "https://example.com/team-traceability-1.5.0.zip",
      "sha256": "<64 hexadecimal digits for the older archive>",
      "requires": {"speckit_version": ">=0.8.0"},
      "provides": {"templates": 1, "commands": 0, "scripts": 0}
    }
  }
}
```

이는 한 항목의 릴리스 관련 부분이지 완전한 카탈로그가 아닙니다.
각 과거 기록에는 자체 HTTPS 아카이브 URL(또는 개발용 루프백 HTTP)과 64자리 SHA-256
다이제스트가 필요합니다. 다이제스트에는 대소문자를 구분하지 않는 `sha256:`
접두사와 앞뒤 공백이 있을 수 있습니다. 다른 알고리즘 접두사는 거부됩니다.

선택적인 과거 `requires`와 `provides`는 그 릴리스를 기술합니다. 없을 때 현재
릴리스의 값을 상속하지 않습니다. 이름, 설명, 카탈로그 권한 같은 공유 메타데이터는
상속됩니다.
과거의 `requires.extensions`는 매니페스트의 문자열 또는 매핑 의존성 형식을
사용합니다. 과거 기록 안에 `id`, `version`, 중첩된 `releases`, 내부
`_catalog_name` / `_install_allowed` 필드를 넣지 마세요.

버전 키는 PEP 440상 동등한 표기를 포함해 서로 달라야 하며 현재 버전을 반복할 수
없습니다. 중복된 JSON 키는 파싱이 릴리스 기록을 버리기 전에 거부됩니다.

```bash
specify preset info team-traceability --versions
specify preset add team-traceability --version 1.5.0
```

사용 가능한 버전은 현재 릴리스를 먼저, 이어서 과거 버전을 내림차순으로
나열합니다. 정확한 릴리스 선택은 설치된 프리셋을 수정하기 전에 다운로드한
매니페스트의 ID와 버전을 확인합니다.

커뮤니티 카탈로그는 탐색 전용으로 남으며 릴리스 이력을 게시할 필요가 없습니다.
번들 고정 해석은 별개의 기능입니다. 프리셋 릴리스 기록을 추가하는 것만으로는 번들
고정을 설치할 수 있게 되지 않습니다.

## 조회와 실패 동작

카탈로그는 우선순위 오름차순으로 확인됩니다. ID에 대한 최상위 우선순위 항목이
릴리스 메타데이터와 설치 권한을 모두 제공합니다. 더 낮은 우선순위의 소스는 누락된
과거 버전을 제공하거나 탐색 전용 결정을 재정의할 수 없습니다.

정확한 ID 조회에서, 이기는 항목보다 먼저 마주친 형식이 잘못되었거나 지나치게 큰
카탈로그는 해석을 실패시킵니다. 형식이 잘못된 더 낮은 우선순위 소스는 이미 찾은
더 높은 우선순위 항목을 막을 수 없습니다. 객체가 아닌 항목은 자신의 ID에 대한
정확한 조회를 실패시키지만, 전체 카탈로그 검색은 그 항목을 건너뛰고 다른 유효한
항목을 유지합니다.

모든 소스에 걸친 검색은 여전히 형식이 잘못된 카탈로그에서 실패합니다. 과거 릴리스
검증은 이기는 항목에 적용됩니다. 가려진 항목의 유효하지 않은 이력은 유효한 더 높은
우선순위 대체 항목을 막지 않습니다.
접근할 수 없는 소스는 다른 소스를 읽을 수 있으면 건너뛸 수 있습니다. 모든 소스가
실패하면 `info --versions`를 포함한 조회는 버전이 없다고 주장하는 대신 가져오기
오류를 보고합니다.

## 캐시와 비공개 소스

카탈로그는 `.specify/presets/.cache/`에 한 시간 동안 캐시됩니다. 기본 카탈로그는
레거시 `catalog.json`과 `catalog-metadata.json` 파일명을 유지하고, 다른 소스는
URL 해시 캐시 파일명을 사용합니다. 캐시 페이로드는 네트워크 응답과 같은 형태
검증을 거칩니다. 읽을 수 없거나 유효하지 않은 캐시 콘텐츠는 가져오기를
트리거합니다.

비공개 카탈로그와 아카이브 요청은 `~/.specify/auth.json`의 공유 선택형 인증 설정을
사용합니다. 토큰 환경 변수만으로는 인증이 활성화되지 않습니다. 자격 증명을 받도록
승인된 호스트만 설정하세요. 인증 레퍼런스가 GitHub CLI, 토큰, 엔터프라이즈, 그
밖의 지원되는 제공자를 다룹니다. 카탈로그 JSON이나 소스 URL에 자격 증명을 넣지
마세요.

## 함께 보기

- [프리셋 개요](overview.md)
- [카탈로그 CLI 명령](cli.md#catalog-management)
- [매니페스트 API](api.md)
- [게시 및 커뮤니티 제출](publishing.md)
- [컴포넌트 공통 카탈로그 안내](../overview/catalogs.md)
- [카탈로그 및 다운로드 인증](../reference/authentication.md)
- [커뮤니티 프리셋](../community/presets.md)
