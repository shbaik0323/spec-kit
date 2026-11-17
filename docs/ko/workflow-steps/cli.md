# 워크플로 스텝 CLI 레퍼런스

`specify workflow step` 그룹은 초기화된 Spec Kit 프로젝트에서 사용자 정의 스텝
패키지와 그 카탈로그를 관리합니다. 내장 타입은 CLI와 함께 제공되며 별도로 설치할
필요가 없습니다.

<a id="step-management"></a>

## 스텝 관리

사용자 정의 스텝에는 실행 가능한 Python이 들어 있습니다. 설치하고 사용하기 전에 소스를 검토하세요. 패키지 계약은 [사용자 정의 스텝 패키지 개발하기](developing.md#custom-step-packages)를 참고하세요.

```bash
specify workflow step list
specify workflow step search [query]
specify workflow step info <step_id>
specify workflow step info <step_id> --versions
```

`list`는 설치된 사용자 정의 스텝 타입을 보여 주고, `search`는 활성 스텝 카탈로그를 검색하며, `info`는 설치된 메타데이터 또는 카탈로그 메타데이터와 소스 출처를 보여 줍니다. `--versions`는 이기는 카탈로그의 릴리스를 나열합니다.

### 로컬 디렉터리에서 설치

```bash
specify workflow step add my-step --dev /path/to/my-step
```

`--dev`는 완전한 패키지인 **디렉터리**를 받습니다(아카이브도, 단독 `step.yml`도
아님). 카탈로그, 서버, 네트워크가 필요 없으므로 지원되는 로컬 작성 루프가 됩니다.

```bash
specify workflow step add my-step --dev ./my-step
specify workflow step list
specify workflow step info my-step
# ./my-step을 편집한 다음 설치된 사본을 교체:
specify workflow step add my-step --dev ./my-step --force
specify workflow step remove my-step
```

### 아카이브 URL에서 설치

```bash
specify workflow step add my-step --from https://example.com/my-step.zip
```

`--from`은 `.zip`, `.tar.gz`, `.tgz` 아카이브를 받습니다(단독 `step.yml` URL은
패키지가 **아닙니다**). 아카이브는 `step.yml`과 `__init__.py`를 루트 또는 정확히
하나의 최상위 디렉터리 아래에 둘 수 있습니다. 관련 없는 최상위 형제 항목은
거부됩니다. 스텝 패키지에는 실행 가능한 Python이 들어 있으므로, 직접 URL 설치는
네트워크 요청 전에 기본 거부 방식의 신뢰 확인을 표시합니다. 거절하면 요청도 오류도
없이 취소됩니다. HTTPS가 필요하고(HTTP는 루프백 호스트에만 허용), 리디렉션은
안전하게 유지되어야 하며, 다운로드에는 크기 제한이 있습니다.

### 카탈로그에서 설치

```bash
specify workflow step add my-step
```

카탈로그 설치는 활성 스텝 카탈로그에서 개별 파일 URL을 해석한 다음 `--dev` 및
`--from`과 같은 검증 및 커밋 경로를 거칩니다.
탐색 전용 카탈로그로부터는 설치할 수 없습니다.

사용자 정의 스텝 타입을 탐색 전용 커뮤니티 카탈로그에 제출하려면
[커뮤니티 워크플로 스텝](../community/workflow-steps.md)을 따르세요. 제출은 태그에
고정된 개별 파일 URL과 파일별 다이제스트를 사용합니다. 메인테이너는 제출된 코드가
아니라 메타데이터와 배포 근거를 검증합니다.

카탈로그 이력과 정확한 버전 선택은 [스텝 릴리스](catalogs.md#step-releases)를 참고하세요.

### 교체와 force

```bash
specify workflow step add my-step --dev ./my-step --force
specify workflow step add my-step --from https://example.com/my-step.zip --force
```

`--force`는 기존 설치를 건드리기 전에 먼저 대체본을 스테이징하고 검증하며, 등록된
설치와 남아 있는 미등록 디렉터리를 모두 교체할 수 있습니다. 검증 및 스테이징
실패는 이전 패키지를 그대로 둡니다. 이전 디렉터리 제거가 실패하면 대체본은
게시되지 않습니다. 이전 디렉터리가 제거된 뒤 대체본 게시나 레지스트리 갱신이
실패하면 설치가 불완전한 상태로 남을 수 있습니다. 원래 소스와 `--force`로 명령을
다시 실행해 재설치하세요. 자동 롤백은 시도되지 않습니다.

### 나열, 실행, 제거

설치된 사용자 정의 스텝은 `specify workflow step list`에 나타나며
`workflow add`, `workflow run`, `workflow resume`에 의해 자동으로 로드됩니다.
제거하려면 다음을 사용하세요.

```bash
specify workflow step remove my-step
```

<a id="catalog-management"></a>

## 카탈로그 관리

스텝 카탈로그는 워크플로 카탈로그와 독립적입니다.

### 카탈로그 나열

```bash
specify workflow step catalog list
```

모든 활성 스텝 카탈로그 소스를 보여 줍니다.

### 카탈로그 추가

```bash
specify workflow step catalog add <url> [--name <name>]
```

| 옵션 | 설명 |
| --- | --- |
| `--name <name>` | 카탈로그의 선택적 이름 |

사용자 정의 카탈로그 URL을 프로젝트의 `.specify/step-catalogs.yml`에 추가합니다.
새 항목은 설치를 허용하며 기존 항목 다음의 우선순위를 받습니다.
검증한 소스만 추가하세요. 탐색 전용 소스는 카탈로그 설정을 편집해 구성하세요.

같은 스텝 카탈로그 URL을 같은 이름으로 다시 추가하면 설정 변경 없이 성공합니다.
다른 이름은 거부됩니다.

### 카탈로그 제거

```bash
specify workflow step catalog remove <index>
```

우선순위 순으로 정렬된 `catalog list`의 인덱스로 프로젝트 카탈로그를 제거합니다.
`SPECKIT_STEP_CATALOG_URL`이 제공하는 소스는 이 방법으로 제거할 수 없습니다.
프로젝트 소스를 관리하려면 그 변수의 설정을 해제하세요.

## 함께 보기

- [스텝 개요](overview.md)
- [내장 스텝 타입](types.md)
- [패키지 개발과 검증](developing.md)
- [Python API 레퍼런스](api.md)
- [카탈로그 스키마와 신뢰](catalogs.md)
- [워크플로 실행 CLI](../workflows/cli.md)
