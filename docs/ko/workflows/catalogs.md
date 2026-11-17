# 워크플로 카탈로그

워크플로 카탈로그는 `search`와 `add`가 워크플로를 찾는 위치를 제어합니다. 카탈로그는 우선순위 순으로 확인됩니다.

워크플로 카탈로그는 가져온 JSON과 신선도 메타데이터를 `.specify/workflows/.cache/`에
한 시간 동안 캐시합니다. 캐시 I/O는 심볼릭 링크된 디렉터리 구성 요소나 캐시 파일을
따라가지 않습니다. 쓰기는 참조 대상을 잘라 내는 대신 캐시 파일을 원자적으로
교체하므로, 기존의 캐시 파일 심볼릭 링크는 그 대상을 바꾸지 않고 교체될 수
있습니다. 캐시를 안전하게 쓸 수 없으면 경고가 기록되고 가져온 카탈로그를 사용해
탐색이 계속됩니다.

> **프로젝트의 `.specify/workflow-catalogs.yml`은 `add`와 `search`가 여러분이 선택하지 않은 카탈로그를 가리키게 할 수 있습니다.** 낯선 프로젝트의 워크플로를 실행하기 전에 `specify workflow catalog list`를(그리고 그 스텝이 끌어올 수 있는 스텝 카탈로그에 대해서는 `specify workflow step catalog list`를) 실행하세요. 프로젝트가 그 설정을 제공한다는 것은 그 워크플로나 스텝이 검증되었다는 증거가 아닙니다. 메인테이너는 워크플로 코드를 감사하지 않습니다. strict 스텝을 포함해 셸 스텝의 `run`과 `env`를 직접 검토하세요([셸 실행 검토하기](../workflow-steps/types.md#reviewing-shell-execution)와 [워크플로는 누가 유지 관리하나요?](overview.md#who-maintains-workflows) 참고).

## 탐색과 신뢰

워크플로 카탈로그는 `workflow.yml` 파일이나 완전한 패키지를 공개합니다. 내장 소스에는 `default`(설치 허용)와 `community`(탐색 전용)가 있습니다. 사용자는 설치하고 사용하기 전에 소스를 검증해야 합니다. 메인테이너는 컴포넌트 코드를 감사하거나 보증하지 않습니다.

명시적인 직접 URL 설치는 카탈로그 조회 없이 소스를 선택합니다. 그렇다고 검토되지 않은 콘텐츠가 안전해지지는 않습니다.

## 카탈로그 설정

| 수준 | 워크플로 |
| --- | --- |
| 환경 변수 오버라이드 | `SPECKIT_WORKFLOW_CATALOG_URL` |
| 프로젝트 | `.specify/workflow-catalogs.yml` |
| 사용자 | `~/.specify/workflow-catalogs.yml` |
| 기본값 | 공식 및 커뮤니티 워크플로 카탈로그 |

처음으로 사용 가능한 수준이 추가 레이어가 아니라 스택 전체를 제공합니다. 환경 변수 오버라이드는 설치 허용 소스 하나를 선택합니다. 빈 설정은 다음 수준으로 넘어가고, 형식이 잘못된 YAML과 유효하지 않은 설정 URL/우선순위는 오류를 보고합니다.

```yaml
catalogs:
  - name: my-org
    url: https://catalog.example.com/workflows.json
    priority: 5
    install_allowed: true
    description: Our vetted workflows
```

`url`은 로컬 개발을 포함해 HTTPS가 필요합니다. 일반 HTTP는 네트워크 접근 전에
거부됩니다. 숫자 우선순위가 낮은 쪽이 이기며, 동률이면 첫 번째 소스가 이깁니다.
손으로 작성한 설정에서 `install_allowed`의 기본값은 false입니다.

`specify workflow catalog add`는 설치가 허용되고 기존 항목 다음의 우선순위를 가진 프로젝트 항목을 만듭니다. 프리셋 방식의 `--install-allowed`나 `--priority` 플래그를 노출하지 않습니다. 검증한 소스만 추가하세요. 탐색 전용 소스를 의도한다면 설정을 편집하세요. 프로젝트 스택을 추가하면, 기본값을 명시적으로 포함하지 않는 한 암묵적인 기본값이 대체됩니다.

### 레거시 HTTP 소스 마이그레이션

오래된 설정에는 더 이상 허용되지 않는 루프백 HTTP URL이 들어 있을 수 있습니다.
레거시 프로젝트 소스는 `specify workflow catalog remove <index>`로 제거하세요.
제거는 소스 URL을 가져오거나 검증하지 않습니다. 인덱스는 우선순위 오름차순을
따르고, 동률은 YAML 선언 순서로 정해지며, URL이 없는 항목은 건너뜁니다.
`catalog list`는 여전히 유효하지 않은 URL을 거부하므로, 목록을 볼 수 없을 때는
`.specify/workflow-catalogs.yml`을 살펴 인덱스를 판단하세요.
레거시 HTTP 소스가 남아 있는 동안에도 다른 항목을 제거할 수 있습니다.

또는 프로젝트 소스는 `.specify/workflow-catalogs.yml`을, 사용자 소스는
`~/.specify/workflow-catalogs.yml`을 편집하세요. 소스의 `url`을 동작하는 HTTPS
카탈로그 URL로 바꾸거나, `catalogs`에서 그 항목 전체를 제거하세요.
로컬 HTTPS 서버에는 Python의 TLS 클라이언트가 신뢰하는 인증서가 필요합니다.
브라우저나 운영 체제의 신뢰만으로는 충분하지 않을 수 있습니다. `SSL_CERT_FILE`
설정은 [TLS 인증서](../reference/authentication.md#tls-certificates)를 참고하세요.
URL 스킴만 바꾼다고 HTTPS가 활성화되지는 않습니다.

비어 있거나 삭제된 설정은 다음 수준으로 넘어갑니다. 더 높은 수준이 스택을
제공하지 않으면 내장 기본값이 적용됩니다. `SPECKIT_WORKFLOW_CATALOG_URL`이 여전히
HTTP 소스를 선택한다면 그것도 갱신하거나 설정을 해제하세요.

## 워크플로 항목 스키마

카탈로그 JSON에는 `schema_version`과 워크플로 ID를 키로 하는 `workflows` 매핑이 있습니다. 항목은 `id`, `name`, `description`, `author`, `version`, `url`, 그리고 선택적인 `sha256`, `requires`, `repository`, `license`, `tags`, 타임스탬프를 담습니다. 다운로드 대상은 원시 YAML 또는 완전한 워크플로 아카이브일 수 있습니다. `bundled: true`로 표시된 퍼스트파티 항목은 네트워크 다운로드가 아니라 패키지된 정의를 사용합니다.

<a id="workflow-releases"></a>

## 워크플로 릴리스

카탈로그 항목은 현재 릴리스의 `version`, `url`, 선택적인 `sha256`, 선택적인
`requires`를 최상위에 유지합니다. 선택적인 `releases` 매핑은 버전을 지정하지 않은
`add`, `search`, `info`, `update`가 선택하는 것을 바꾸지 않으면서 과거 버전을
공개합니다.

```json
{
  "id": "example",
  "version": "2.0.0",
  "url": "https://example.com/example-2.0.0.zip",
  "releases": {
    "1.0.0": {
      "url": "https://example.com/example-1.0.0.zip",
      "sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "requires": {"speckit_version": ">=1.0.0"}
    }
  }
}
```

각 과거 릴리스에는 자체 URL과 SHA-256 다이제스트가 필요합니다. `requires`는 선택
사항이며, 있으면 다운로드한 워크플로 정의와 일치해야 합니다.
공개되는 버전은 워크플로 정의의 `X.Y.Z` 버전 형식을 사용합니다. `--version`은
공개된 `1.0.0` 릴리스를 선택할 때 `v1.0` 같은 동등한 표기도 받습니다.
요청한 버전은 그 워크플로를 제공하는 최상위 우선순위 카탈로그에 존재해야 합니다.
없는 버전은 다른 소스로 폴백하지 않으며, 탐색 전용 카탈로그로부터는 설치할 수
없습니다. 다운로드한 워크플로 ID, 버전, 선언된 다이제스트는 설치 전에 검증됩니다.
`--version`은 로컬 경로, 직접 URL, `--from` 설치에는 적용되지 않습니다.

## 해석과 실패

워크플로 조회는 정확한 ID에 대한 조회를 포함해 모든 활성 카탈로그를 병합합니다.
ID를 찾은 뒤에도 멈추지 않습니다. 더 높은 우선순위 항목이 더 낮은 우선순위 항목을
덮어쓰도록 소스는 우선순위 역순으로 처리됩니다. 다른 소스가 그 ID를 제공하더라도
각 소스는 가져오거나 캐시에서 읽힐 수 있습니다.
이길 것으로 예상되는 소스만이 아니라 모든 활성 소스를 검증하세요.

이기는 항목이 메타데이터, 릴리스, 설치 정책을 함께 제공합니다. 더 낮은 우선순위
항목은 누락된 버전을 제공하거나 이기는 탐색 전용 결정을 재정의할 수 없습니다.
하지만 가져오기에 실패한 소스는 항목을 기여하지 않으므로, 우선하는 소스를 사용할
수 없을 때는 더 낮은 우선순위 소스가 그 ID를 제공할 수 있습니다. 캐시 폴백 후에도
남는 가져오기 실패는 다른 소스가 성공하면 건너뜁니다. 설정된 모든 카탈로그가
실패하면 조회는 오류를 보고합니다.

가져온 카탈로그는 JSON 객체여야 합니다. 그 `workflows` 컬렉션은 매핑 또는 목록
형식을 받습니다. 지원되지 않는 컬렉션 형태와 매핑이 아닌 항목은 건너뜁니다. 이렇게
무시된 내용은 가져오기 실패로 계산되지 않으므로, 성공적으로 가져온 객체가 항목을
하나도 기여하지 않을 수 있고 조회는 구조 검증 오류가 아니라 결과 없음을 반환할 수
있습니다. 릴리스 메타데이터는 별도로 검증되며 여전히 오류를 낼 수 있습니다.

## 캐시와 비공개 소스

워크플로 카탈로그 캐시는 `.specify/workflows/.cache/` 아래에 있으며 수명은 한 시간입니다. 페이로드와 신선도 파일은 URL 해시를 키로 합니다. 사용할 수 없는 캐시 항목은 가져오기를 트리거합니다. 안전하지 않은 캐시 쓰기는 가져온 탐색 결과를 버리지 않고 경고를 기록합니다.

비공개 카탈로그와 패키지 다운로드는 `~/.specify/auth.json`의 공유 선택형 인증을 사용합니다. 토큰 환경 변수만으로는 활성화되지 않습니다. 승인된 대상 호스트만 설정하세요. 카탈로그 JSON이나 URL에 자격 증명을 절대 넣지 마세요.

## 함께 보기

- [카탈로그 CLI 명령](cli.md#catalog-management)
- [워크플로 게시하기](publishing.md)
- [커뮤니티 워크플로](../community/workflows.md)
- [스텝 카탈로그](../workflow-steps/catalogs.md)
- [컴포넌트 공통 카탈로그](../overview/catalogs.md)
- [다운로드 인증](../reference/authentication.md)
- [셸 소스 검토](../workflow-steps/types.md#reviewing-shell-execution)
