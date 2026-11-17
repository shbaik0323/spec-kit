# 번들 카탈로그

번들 카탈로그는 번들 매니페스트의 위치를 알려 주고 탐색할 수 있는 구성을
설명합니다. 그 매니페스트에 필요한 확장, 프리셋, 워크플로, 스텝 카탈로그를
제공하지는 않습니다. 그런 컴포넌트 카탈로그는 따로 등록하고 검증하세요.

## 신뢰와 설치 정책

> [!IMPORTANT]
> 설치 전에 번들 소스와 그 컴포넌트 카탈로그를 모두 검증하세요.
> 프로젝트는 `.specify/` 아래에 번들 카탈로그와 설치 허용 컴포넌트 카탈로그를
> 제공할 수 있습니다. 그것이 있다고 해서 누군가 검토했다는 증거는 아닙니다.

직접 추가하지 않은 것을 포함해 해당하는 모든 소스를 확인하세요.

```bash
specify bundle catalog list
specify extension catalog list
specify preset catalog list
specify workflow catalog list
specify workflow step catalog list
```

낯선 소스는 검토하기 전까지 검증되지 않은 것으로 취급하세요. 워크플로
컴포넌트의 경우 활성 오버레이를 포함해 셸 `run` 및 `env` 필드와 참조된 코드를
확인하세요. `strict: true` 스텝조차 환경 바인딩을 통해 실행 파일 탐색이나 런타임
로딩을 바꿀 수 있습니다.
[셸 실행 검토하기](../workflow-steps/types.md#reviewing-shell-execution)를
참고하세요.

| 정책 | 효과 |
| --- | --- |
| `install-allowed` | 이 소스로부터의 설치/업데이트를 허용 |
| `discovery-only` | 검색과 조회는 허용하지만 설치/업데이트는 거부 |

`verified` 플래그는 카탈로그 메타데이터이지, 설치 정책이나 보안 보장이 아닙니다.
커뮤니티 항목은 검증되지 않은 것이며, 메인테이너는 그 코드를 검토, 감사, 보증,
지원하지 않습니다. 번들 설치는 동반 카탈로그를 자동으로 등록하거나 신뢰하지
않습니다.

## 내장 소스

| 소스 ID | URL | 우선순위 | 정책 |
| --- | --- | --- | --- |
| `default` | `builtin://default` | `1` | `install-allowed` |
| `community` | `builtin://community` | `20` | `discovery-only` |

기본 카탈로그는 `bundles/catalog.json`이고, 커뮤니티 제출은
`bundles/catalog.community.json`에 나타납니다. 둘 다 온라인에서 저장소로부터
가져옵니다. 오프라인 탐색은 패키지된 휠 스냅샷을 사용합니다. 내장 카탈로그의
일시적인 가용성 실패는 경고와 함께 그 스냅샷으로 폴백합니다. 형식이 잘못된
콘텐츠나 보안 검증 실패는 조용히 폴백하지 않습니다.

번들 매니페스트의 `download_url`은 여전히 네트워크 접근이 필요합니다. 패키지된
카탈로그 스냅샷은 로컬 페이로드 저장소가 아닙니다.

## 소스 설정

초기화된 프로젝트에서, 소스를 검증한 뒤:

```bash
specify bundle catalog add https://example.com/bundles.json \
  --id team --priority 10 --policy install-allowed
specify bundle catalog list
specify bundle catalog remove team
```

CLI는 `.specify/bundle-catalogs.yml`을 기록합니다.

```yaml
schema_version: "1.0"
catalogs:
  - id: team
    url: https://example.com/bundles.json
    priority: 10
    install_policy: install-allowed
```

사용자 설정은 `~/.specify/bundle-catalogs.yml`에서 같은 형태를 사용합니다.
CLI의 add/remove 작업은 프로젝트 범위에만 기록합니다. 두 리더 모두 호환성을 위해
스키마 버전 누락을 허용하고 지원되지 않는 메이저 버전을 거부합니다.

소스 URL은 HTTPS, `file://`, 로컬 경로, 알려진 `builtin://` URL일 수 있습니다.
일반 HTTP는 localhost를 포함해 네트워크 접근 전에 거부됩니다. 추가된 로컬 경로는
절대 경로로 저장됩니다. 이 유연성은 카탈로그 파일에 적용되며, 그 번들 다운로드
URL에는 적용되지 않습니다.

### 우선순위

유효 스택은 두 단계로 결정됩니다.

1. 소스를 ID로 병합합니다. 프로젝트 범위 소스가 사용자 범위의 같은 ID를 대체하고,
   사용자 범위가 내장 범위의 같은 ID를 대체합니다.
2. 남은 소스를 숫자 우선순위로, 그다음 소스 ID로 정렬합니다. 우선순위 숫자가
   낮은 쪽이 이깁니다.

서로 다른 ID에 대해서는 범위가 우선순위보다 앞서지 않습니다. 추가된 소스는 기본
우선순위가 `10`으로, `20`인 커뮤니티 소스보다 앞서지만 `1`인 기본 소스보다는
뒤입니다. 내장 소스를 그 ID로 재정의하거나, 의도적으로 적절한 우선순위를
선택하세요.

번들 ID를 담은 첫 번째 소스가 이깁니다. 과거 릴리스 누락, 소스 불일치, 탐색 전용
정책은 더 낮은 우선순위 카탈로그로의 폴백을 일으키지 않습니다. 번들 참조의
`source`는 이 승자를 단언할 뿐, 소스를 선택하지 않습니다.

기존 URL을 일치하는 ID/설정으로 다시 추가하면 아무 일도 일어나지 않습니다.
충돌하는 URL/ID 설정은 실패합니다. 내장 소스는 제거할 수 없지만, 내장 ID에 대한
프로젝트 재정의는 제거해 더 낮은 범위의 소스를 복원할 수 있습니다.

## 카탈로그 페이로드

카탈로그 파일은 번들 ID를 키로 하는 `bundles` 매핑을 가진 JSON 객체입니다.

```json
{
  "schema_version": "1.0",
  "bundles": {
    "team-setup": {
      "id": "team-setup",
      "name": "Team setup",
      "version": "1.2.0",
      "role": "developer",
      "description": "A curated team setup",
      "author": "Example team",
      "license": "MIT",
      "repository": "https://github.com/example/team-setup",
      "download_url": "https://github.com/example/team-setup/releases/download/v1.2.0/team-setup-1.2.0.zip",
      "requires": {"speckit_version": ">=0.9.0"},
      "provides": {"extensions": 1, "presets": 0, "steps": 0, "workflows": 1},
      "tags": ["developer", "bugfix"],
      "verified": false
    }
  }
}
```

항목의 `id`는 그것을 감싸는 키와 일치해야 합니다. `requires`와 `provides`는
존재하고 null이 아니면 매핑이어야 하고, `tags`는 문자열 목록이어야 하며,
`verified`는 실제 불리언이어야 합니다. `provides`는 매니페스트의 컴포넌트 목록이
아니라 컴포넌트 수를 기술합니다. 지원되지 않는 메이저 버전을 가진 카탈로그 스키마
버전과 중복된 JSON 키는 거부됩니다.

아티팩트 바이트를 검증하려면 `sha256` 다이제스트를 사용하세요. 커뮤니티 제출
자동화는 실제 아카이브로부터 계산한 다이제스트를 기록합니다. 제출된 체크섬이
일치하지 않으면 검증이 실패합니다. 범용 현재 항목 리더는 여전히 다이제스트 없는
항목을 허용합니다. 과거 릴리스에는 자체 다이제스트가 필요합니다.

카탈로그의 번들 다운로드 URL은 HTTPS, 또는 localhost의 HTTP를 사용해야 하며, 번들
매니페스트나 루트 수준 `bundle.yml`을 담은 ZIP으로 해석되어야 합니다. 가져온
매니페스트의 ID와 버전은 선택된 카탈로그 항목과 일치해야 합니다. 로컬 경로와
`file://`은 번들 다운로드 URL로 허용되지 않습니다. 로컬 번들 소스는
`bundle install`에 직접 전달하세요.

커뮤니티 제출에는 더 좁은 게시 계약이 있습니다. 버전이 지정되고 태그에 고정된
GitHub 릴리스 ZIP 자산입니다. 게시 가이드를 참고하세요.

## 릴리스 이력

항목은 현재 릴리스와 함께 승인된 과거 릴리스를 추가할 수 있습니다.

```json
{
  "id": "team-setup",
  "version": "1.2.0",
  "download_url": "https://example.com/team-setup-1.2.0.zip",
  "releases": {
    "1.1.0": {
      "download_url": "https://example.com/team-setup-1.1.0.zip",
      "sha256": "<64-character SHA-256 digest>"
    }
  }
}
```

이는 항목의 일부이지 완전한 카탈로그가 아닙니다. 사용하기 전에 다이제스트 자리
표시자를 교체하세요. 각 과거 기록에는 자체 HTTPS(또는 localhost HTTP)
`download_url`과 64자 16진수 SHA-256 다이제스트가 필요합니다.
다이제스트에는 대소문자를 구분하지 않는 `sha256:` 접두사와 앞뒤 공백이 있을 수
있습니다. 다른 알고리즘 접두사는 거부됩니다.

이력 키는 PEP 440으로 비교 가능한 엄격한 SemVer 값이어야 합니다. `1.0.0-rc.1`과
`1.0.0-rc1` 같은 동등한 표기를 포함해 현재 버전이나 서로와 중복될 수 없습니다.
과거 기록에는 `id`, `version`, `releases`가 들어갈 수 없습니다.

이름, 설명, 작성자 같은 공유 메타데이터는 재정의하지 않는 한 상속됩니다. 릴리스별
`requires`, `provides`, `verified`는 상속되지 않습니다. 각 과거 기록에
제공하세요. 그렇지 않으면 그 릴리스에는 요구 사항도, 제공 수도 없고 `community`
신뢰를 갖습니다.

```bash
specify bundle info team-setup --versions
specify bundle install team-setup --version 1.1.0
```

`v1.1.0`과 `1.1.0` 같은 동등한 버전 표기는 같은 릴리스를 선택합니다. 카탈로그 키
표기는 표시용일 뿐입니다. 정확한 번들 릴리스 선택은 가져온 매니페스트의 컴포넌트
고정을 절대 바꾸지 않습니다.
`bundle update`는 항상 현재 릴리스를 선택합니다.

이력은 카탈로그 소스가 로드될 때 검증됩니다. 형식이 잘못된 이력은 그 소스를
참조하는 모든 작업을 실패시킵니다. ID 조회는 승자를 찾은 뒤 형식이 잘못된 더 낮은
우선순위의 소스를 살펴보지 않고 넘어갈 수 있습니다. 검색은 모든 소스를 로드하므로
가려진 잘못된 항목도 검색을 실패시킵니다.

기존의 단일 릴리스 카탈로그는 여전히 유효합니다. 퍼스트파티 및 커뮤니티 번들
카탈로그는 현재 번들당 하나의 릴리스만 공개하며 과거 버전을 보존하지 않습니다.

## 함께 보기

- [번들 CLI 레퍼런스](cli.md)
- [매니페스트와 소스 단언](api.md)
- [게시 및 커뮤니티 제출](publishing.md)
- [커뮤니티 번들](../community/bundles.md)
- [공통 카탈로그 개념](../overview/catalogs.md)
