# 카탈로그

카탈로그는 Specify가 통합, 확장, 프리셋, 워크플로, 워크플로 스텝 타입, 번들을
탐색할 수 있게 해 주는 인덱스입니다. Spec Kit은 가능한 경우 커뮤니티 카탈로그를
포함한 내장 카탈로그를 제공하며, 프로젝트나 사용자가 비공개 카탈로그 소스를
추가할 수 있게 합니다.

카탈로그는 모든 컴포넌트 유형이 공유하는 단일 레지스트리가 아닙니다. 컴포넌트
유형마다 고유한 카탈로그 스키마, 명령 그룹, 설정, 해석 스택이 있습니다. 여러
컴포넌트 유형을 선별하는 조직은 보통 유형마다 카탈로그 파일을 하나씩 게시합니다.

## 컴포넌트 탐색

검색은 해당 컴포넌트 유형의 모든 활성 카탈로그를 사용합니다.

```bash
specify integration search <query>
specify extension search <query>
specify preset search <query>
specify workflow search <query>
specify workflow step search <query>
specify bundle search <query>
```

공개된 릴리스를 살펴보려면 해당하는 버전 보기를 사용하세요.

```bash
specify integration info <id> --versions
specify extension info <id> --versions
specify preset info <id> --versions
specify workflow info <id> --versions
specify workflow step info <id> --versions
specify bundle info <id> --versions
```

일반 `info` 출력과 `--versions`가 보여 주는 세부 정보는 컴포넌트 유형에 따라
다릅니다. 버전 보기는 현재 및 과거의 카탈로그 릴리스를 나열하고, 지원되는 경우
이기는 소스를 설치 가능 또는 탐색 전용으로 표시합니다. 탐색 전용 릴리스는 버전
목록에 나타나더라도 ID로는 설치할 수 없습니다. 일반 `info` 출력에 의존하지 말고
컴포넌트의 `catalog list` 명령으로 활성 소스와 설치 정책을 확인하세요.

[통합](../community/integrations.md),
[확장](../community/extensions.md), [프리셋](../community/presets.md),
[워크플로](../community/workflows.md),
[워크플로 스텝 타입](../community/workflow-steps.md),
[번들](../community/bundles.md)의 커뮤니티 페이지도 둘러볼 수 있습니다.

## 카탈로그 소스 조회 및 관리

낯선 프로젝트에서 컴포넌트를 설치하기 전에 활성 카탈로그 스택을 나열하세요.

| 컴포넌트 유형 | 활성 카탈로그 나열 | 상세 레퍼런스 |
| --- | --- | --- |
| 통합 | `specify integration catalog list` | [통합](../integrations/cli.md#catalog-management) |
| 확장 | `specify extension catalog list` | [확장](../extensions/catalogs.md) |
| 프리셋 | `specify preset catalog list` | [프리셋 카탈로그](../presets/catalogs.md) |
| 워크플로 | `specify workflow catalog list` | [워크플로 카탈로그](../workflows/catalogs.md) |
| 워크플로 스텝 | `specify workflow step catalog list` | [스텝 카탈로그](../workflow-steps/catalogs.md) |
| 번들 | `specify bundle catalog list` | [번들 카탈로그](../bundles/catalogs.md) |

각 명령 그룹은 `catalog add`와 `catalog remove`도 제공합니다. 옵션, 식별자, 설치
정책, 설정 파일명은 컴포넌트 유형마다 다르므로, 카탈로그 스택을 바꾸기 전에 링크된
레퍼런스를 참고하세요.

카탈로그 범위와 우선순위는 두 가지 모델을 사용합니다.

- **통합, 확장, 프리셋, 워크플로, 워크플로 스텝**은 설정 레이어 하나를
  선택합니다. 환경 변수 오버라이드는 모든 것을 대체합니다. 그렇지 않으면 프로젝트
  카탈로그 파일이 사용자 및 내장 레이어를 대체하고, 사용자 파일이 내장 레이어를
  대체하며, 내장 카탈로그는 더 높은 레이어가 설정되지 않았을 때만 사용됩니다.
  선택된 레이어 안에서는 항목이 숫자 우선순위로 정렬됩니다. 따라서 첫 번째
  프로젝트 카탈로그를 추가하면, 내장 소스도 프로젝트 파일에 나열하지 않는 한 활성
  스택에서 내장 소스가 제거됩니다.
- **번들**은 내장, 사용자, 프로젝트 소스를 소스 ID로 병합합니다. 더 높은 범위의
  정의가 같은 ID를 가진 더 낮은 범위의 소스를 대체한 다음, 남은 모든 소스가 숫자
  우선순위로 정렬됩니다. 프로젝트 소스가 이름이 다른 내장 소스보다 자동으로
  우선하지는 않습니다.

성공적으로 로드된 카탈로그 중에서는 식별자를 제공하는 더 높은 우선순위 항목이
이깁니다. 일부 컴포넌트 리더는 일반적인 가져오기 실패를 건너뛰고 남은 소스로
계속 진행하므로, 접근할 수 없는 더 높은 우선순위 소스 때문에 더 낮은 우선순위
소스가 같은 식별자를 해석하게 될 수 있습니다. 소스 우선순위를 실패 시 차단되는
(fail-closed) 보안 통제로 취급하지 마세요.

<a id="discovery-and-trust"></a>

## 탐색과 신뢰

카탈로그에 등재되었다는 것은 감사나 보증이 아닙니다. 카탈로그는 다음 중 하나일 수
있습니다.

- **설치 허용(install-allowed)**: 컴포넌트 ID로 설치하는 것을 신뢰하는 소스입니다.
- **탐색 전용(discovery-only)**: 평가를 위해 검색할 수 있지만, 그 카탈로그를 통한
  ID 설치는 차단됩니다.

탐색 전용 정책은 지원되는 `--from` URL이나 `--dev` 경로처럼 별도로 요청한 직접
소스 설치를 차단하지 않습니다. 직접 설치 경로를 사용하기 전에 그 소스를
독립적으로 검토하세요.

커뮤니티 카탈로그는 탐색을 위한 표면이며 탐색 전용으로 유지해야 합니다. 사용하기
전에 컴포넌트의 소스와 릴리스 아티팩트를 검토하세요. 검토한 컴포넌트를 팀 전체에서
재사용할 수 있게 하려면, 검증되지 않은 공개 카탈로그를 설치 허용으로 바꾸지 말고
검증한 항목을 조직이 통제하는 카탈로그로 복사하세요.

프로젝트 카탈로그 설정도 신뢰할 수 없는 입력입니다. 저장소는 `.specify/` 아래에
카탈로그 소스를 추가하거나 우선순위를 매길 수 있지만, 그런 설정이 있다고 해서 그
소스나 컴포넌트가 승인되었다는 뜻은 아닙니다. 낯선 프로젝트에서 컴포넌트를
설치하거나 실행하기 전에 관련 `catalog list` 명령을 실행하고 설치 허용 소스를
모두 확인하세요.

## 비공개 카탈로그 호스팅

1. 설치 허용 소스를 등록하기 전에 항목과 릴리스 아티팩트를 검토하고 승인합니다.
2. 컴포넌트 유형을 선택하고 해당 카탈로그 스키마나 게시 가이드를 따릅니다.
3. 대상 사용자가 접근할 수 있는 HTTPS URL에 카탈로그 JSON을 게시합니다.
4. 컴포넌트의 `catalog add` 명령으로 그 URL을 추가합니다.
5. `catalog list`로 결과 소스, 우선순위, 설치 정책을 확인합니다.

`catalog add`의 신뢰 기본값은 컴포넌트 유형마다 다릅니다.

| 컴포넌트 유형 | 추가 시 기본값 |
| --- | --- |
| 통합 | 설치 허용 |
| 확장 | 탐색 전용. 설치를 활성화하려면 `--install-allowed` 필요 |
| 프리셋 | 탐색 전용. 설치를 활성화하려면 `--install-allowed` 필요 |
| 워크플로 | 설치 허용 |
| 워크플로 스텝 | 설치 허용 |
| 번들 | 설치 허용. 제한하려면 `--policy discovery-only` 사용 |

통합, 워크플로, 워크플로 스텝, 번들은 추가하는 즉시 설치가 허용될 수 있으므로,
검토하지 않은 소스를 등록해 두고 나중에 승인하려는 계획은 세우지 마세요.

카탈로그 항목을 작성할 때는 컴포넌트별 게시 안내를 사용하세요.

| 컴포넌트 유형 | 게시 안내 |
| --- | --- |
| 통합 | [통합 카탈로그 기여](../integrations/contributing.md) |
| 확장 | [확장 게시 가이드](../extensions/publishing.md) |
| 프리셋 | [프리셋 게시 가이드](../presets/publishing.md) |
| 워크플로 | [워크플로 게시](../workflows/publishing.md) |
| 워크플로 스텝 | [워크플로 스텝 게시](../workflow-steps/publishing.md) |
| 번들 | [커뮤니티 번들](../community/bundles.md) |

예를 들어 검토를 마친 확장 카탈로그는 다음과 같이 등록할 수 있습니다.

```bash
specify extension catalog add https://catalog.example.com/extensions.json \
  --name approved-extensions \
  --priority 5 \
  --install-allowed
```

프리셋, 통합, 워크플로, 워크플로 스텝, 번들에는 별도의 카탈로그 파일과 명령을
사용하세요. 각 상세 레퍼런스가 지원되는 옵션과 기대되는 항목 형식을 문서화합니다.

## 비공개 및 오프라인 환경

카탈로그나 그 컴포넌트 다운로드에 자격 증명이 필요하다면
[카탈로그 및 다운로드 인증](../reference/authentication.md)을 설정하세요. 인증은
선택형이며 명시적으로 설정한 호스트로 범위가 한정됩니다.

방화벽 뒤에서는 승인된 내부 HTTPS 엔드포인트에 카탈로그와 아티팩트를 호스팅하고
그 엔드포인트를 해당 카탈로그 스택에 추가하세요. 완전히 단절된 환경에서는 승인된
절차를 통해 CLI와 검토한 컴포넌트 아티팩트를 옮기세요. 내장 자산은 네트워크 접근
없이도 계속 사용할 수 있습니다. CLI를 오프라인으로 설치하는 방법은
[엔터프라이즈 / 에어갭 설치](../install/air-gapped.md)를 참고하세요.

외부 통합 어댑터에는 직접 또는 로컬 설치 경로가 없습니다. 설치하려면 설치 허용
통합 카탈로그와 그것이 참조하는 아카이브가 모두 접근 가능한 상태여야 합니다.
[카탈로그로 설치하는 외부 어댑터](../integrations/cli.md#catalog-installed-external-adapters)를
참고하세요.

카탈로그 메타데이터만으로는 오프라인 설치에 충분하지 않습니다. 선택한 모든 릴리스
아티팩트와 번들이 제공하는 모든 의존성도 접근 가능하거나 환경 안으로 옮겨져 있어야
합니다.
