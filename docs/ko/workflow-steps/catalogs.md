# 워크플로 스텝 카탈로그

스텝 카탈로그는 `specify workflow step search`와 `add`가 사용자 정의 스텝 패키지를
찾는 위치를 제어합니다. 워크플로 카탈로그와 독립적입니다.

## 탐색과 신뢰

스텝 카탈로그는 개별 파일 URL을 통해 실행 가능한 Python 패키지를 공개합니다. 내장
소스에는 `default`(설치 허용)와 `community`(탐색 전용)가 있습니다. 사용자는
설치하고 사용하기 전에 소스를 검증해야 합니다. 메인테이너는 컴포넌트 코드를
감사하거나 보증하지 않습니다.

프로젝트의 `.specify/step-catalogs.yml`은 여러분이 선택하지 않은 소스를 선택할 수
있습니다. 사용자 정의 스텝을 설치하거나 로드하기 전에
`specify workflow step catalog list`로 확인하세요. 직접 아카이브 URL은 카탈로그
조회를 건너뛸 뿐 소스 검토를 건너뛰지 않습니다. 직접 URL 설치는 네트워크 요청을
하기 전에 신뢰 확인을 묻습니다.

## 카탈로그 설정

| 수준 | 스텝 타입 |
| --- | --- |
| 환경 변수 오버라이드 | `SPECKIT_STEP_CATALOG_URL` |
| 프로젝트 | `.specify/step-catalogs.yml` |
| 사용자 | `~/.specify/step-catalogs.yml` |
| 기본값 | 공식 및 커뮤니티 스텝 카탈로그 |

처음으로 사용 가능한 수준이 추가 레이어가 아니라 스택 전체를 제공합니다. 환경 변수
오버라이드는 설치 허용 소스 하나를 선택합니다. 빈 설정은 다음 수준으로 넘어가고,
형식이 잘못된 YAML과 유효하지 않은 설정 URL/우선순위는 오류를 보고합니다.

```yaml
catalogs:
  - name: my-org
    url: https://catalog.example.com/steps.json
    priority: 5
    install_allowed: true
    description: Our vetted workflow steps
```

`url`은 로컬 개발을 포함해 HTTPS가 필요합니다. 일반 HTTP는 네트워크 접근 전에
거부됩니다. 숫자 우선순위가 낮은 쪽이 이기며, 동률이면 첫 번째 소스가 이깁니다.
손으로 작성한 설정에서 `install_allowed`의 기본값은 false입니다.

`specify workflow step catalog add`는 설치가 허용되고 기존 항목 다음의 우선순위를
가진 프로젝트 항목을 만듭니다. 프리셋 방식의 `--install-allowed`나 `--priority`
플래그를 노출하지 않습니다. 검증한 소스만 추가하세요. 탐색 전용 소스를 의도한다면
설정을 편집하세요. 프로젝트 스택을 추가하면, 기본값을 명시적으로 포함하지 않는 한
암묵적인 기본값이 대체됩니다.

### 레거시 HTTP 소스 마이그레이션

오래된 설정에는 더 이상 허용되지 않는 루프백 HTTP URL이 들어 있을 수 있습니다.
레거시 프로젝트 소스는 `specify workflow step catalog remove <index>`로 제거하세요.
제거는 소스 URL을 가져오거나 검증하지 않습니다. 인덱스는 우선순위 오름차순을
따르고, 동률은 YAML 선언 순서로 정해지며, URL이 없는 항목은 건너뜁니다.
`step catalog list`는 여전히 유효하지 않은 URL을 거부하므로, 목록을 볼 수 없을 때는
`.specify/step-catalogs.yml`을 살펴 인덱스를 판단하세요.
레거시 HTTP 소스가 남아 있는 동안에도 다른 항목을 제거할 수 있습니다.

또는 프로젝트 소스는 `.specify/step-catalogs.yml`을, 사용자 소스는
`~/.specify/step-catalogs.yml`을 편집하세요. 소스의 `url`을 동작하는 HTTPS
카탈로그 URL로 바꾸거나, `catalogs`에서 그 항목 전체를 제거하세요.
로컬 HTTPS 서버에는 Python의 TLS 클라이언트가 신뢰하는 인증서가 필요합니다.
브라우저나 운영 체제의 신뢰만으로는 충분하지 않을 수 있습니다. `SSL_CERT_FILE`
설정은 [TLS 인증서](../reference/authentication.md#tls-certificates)를 참고하세요.
URL 스킴만 바꾼다고 HTTPS가 활성화되지는 않습니다.

비어 있거나 삭제된 설정은 다음 수준으로 넘어갑니다. 더 높은 수준이 스택을
제공하지 않으면 내장 기본값이 적용됩니다. `SPECKIT_STEP_CATALOG_URL`이 여전히 HTTP
소스를 선택한다면 그것도 갱신하거나 설정을 해제하세요.

## 스텝 항목 스키마

카탈로그 JSON에는 `schema_version`과 `steps` 컬렉션이 있습니다. 항목은 타입과
버전, `step_yml_url`(또는 `url`), `init_url`, 선택적인 `extra_files`, 파일별
`sha256` 다이제스트를 식별합니다. 카탈로그 설치는 `--from`이 사용하는 아카이브가
아니라 개별 패키지 파일을 다운로드합니다.

커뮤니티 제출 가이드가 커뮤니티 항목에 대한 추가 메타데이터와 고정 파일 요구
사항을 정의합니다.

<a id="step-releases"></a>

## 스텝 릴리스

`specify workflow step info <id> --versions`는 이기는 카탈로그의 현재 및 과거
릴리스를 나열합니다. `specify workflow step add <id> --version <v>`는 정확한
카탈로그 릴리스를 선택합니다. `--version`이 없으면 `add`는 공개된 현재 릴리스를
설치합니다. 요청한 버전은 더 낮은 우선순위의 카탈로그로 폴백하지 않으며, 탐색 전용
소스는 설치할 수 없는 상태로 남습니다.
`--version`은 직접 `--dev` 또는 `--from` 설치와 결합할 수 없습니다.

기존의 단일 버전 카탈로그 항목은 계속 동작합니다. 현재 릴리스의 `version`,
`step_yml_url`(또는 `url`), `init_url`, 선택적 파일은 최상위에 유지하세요.
선택적인 `releases` 매핑이 과거 버전을 추가합니다.

```json
{
  "steps": {
    "my-step": {
      "version": "2.0",
      "step_yml_url": "https://example.com/my-step/2.0/step.yml",
      "releases": {
        "1.0": {
          "step_yml_url": "https://example.com/my-step/1.0/step.yml",
          "init_url": "https://example.com/my-step/1.0/__init__.py",
          "extra_files": {"helper.py": "https://example.com/my-step/1.0/helper.py"},
          "sha256": {
            "step.yml": "<64 hex digits>",
            "__init__.py": "<64 hex digits>",
            "helper.py": "<64 hex digits>"
          }
        }
      }
    }
  }
}
```

각 과거 릴리스는 자체 스텝 URL과 다운로드되는 모든 파일의 SHA-256 다이제스트를
제공해야 합니다. `init_url`은 `step.yml`로 끝나는 `step_yml_url`로부터 도출되는
경우 생략할 수 있습니다. 현재 버전을 명시적으로 선택하려면 그 최상위 항목도
마찬가지로 파일별 `sha256` 매핑을 제공해야 합니다.
릴리스별 파일과 요구 사항은 현재 릴리스로부터 상속되지 않습니다. 반복되거나 형식이
잘못된 버전은 거부됩니다. `v1.0`과 `1.0` 같은 동등한 PEP 440 표기는 같은 공개
릴리스를 선택합니다. 다운로드한 `step.yml`은 선택된 스텝 ID와 버전을 선언해야
합니다. 버전을 지정하지 않은 레거시 카탈로그 설치에는 다이제스트나 스텝 버전이
필요하지 않습니다.
스텝 ID마다 한 번에 하나의 버전만 설치할 수 있습니다. 선택한 패키지를 검토한 뒤
이전 설치를 교체하려면 `--force`를 사용하세요.

## 해석과 실패

스텝 조회는 정확한 ID에 대한 조회를 포함해 모든 활성 카탈로그를 병합합니다.
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

가져온 카탈로그는 JSON 객체여야 합니다. 그 `steps` 컬렉션은 매핑 또는 목록 형식을
받습니다. 지원되지 않는 컬렉션 형태와 매핑이 아닌 항목은 건너뜁니다. 이렇게 무시된
내용은 가져오기 실패로 계산되지 않으므로, 성공적으로 가져온 객체가 항목을 하나도
기여하지 않을 수 있고 조회는 구조 검증 오류가 아니라 결과 없음을 반환할 수
있습니다. 스텝 카탈로그는 추가로 중복된 JSON 필드와 목록 형식의 중복 스텝 ID를
거부합니다. 릴리스 메타데이터는 별도로 검증되며 여전히 오류를 낼 수 있습니다.

## 캐시와 비공개 소스

스텝 카탈로그는 가져온 JSON과 신선도 메타데이터를
`.specify/workflows/steps/.cache/`에 한 시간 동안 캐시합니다. 페이로드와 신선도
파일은 URL 해시를 키로 합니다. 사용할 수 없는 캐시 항목은 가져오기를 트리거합니다.
캐시 I/O는 심볼릭 링크된 디렉터리 구성 요소나 캐시 파일을 따라가지 않습니다.
쓰기는 참조 대상을 잘라 내는 대신 캐시 파일을 원자적으로 교체하므로, 기존의 캐시
파일 심볼릭 링크는 그 대상을 바꾸지 않고 교체될 수 있습니다. 안전하지 않은 캐시
쓰기는 가져온 탐색 결과를 버리지 않고 경고를 기록합니다.

비공개 카탈로그와 패키지 다운로드는 `~/.specify/auth.json`의 공유 선택형 인증을
사용합니다. 토큰 환경 변수만으로는 활성화되지 않습니다.
승인된 대상 호스트만 설정하세요. 카탈로그 JSON이나 URL에 자격 증명을 절대 넣지
마세요.

## 함께 보기

- [카탈로그 CLI 명령](cli.md#catalog-management)
- [스텝 게시하기](publishing.md)
- [커뮤니티 스텝 제출](../community/workflow-steps.md)
- [워크플로 카탈로그](../workflows/catalogs.md)
- [컴포넌트 공통 카탈로그](../overview/catalogs.md)
- [다운로드 인증](../reference/authentication.md)
