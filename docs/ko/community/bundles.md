# 커뮤니티 번들

> [!NOTE]
> 커뮤니티 번들은 각 작성자가 독립적으로 만들고 유지 관리합니다. 메인테이너는 제출 메타데이터가 완전하고 올바른 형식인지만 확인할 뿐, **번들 코드나 그것이 설치하는 컴포넌트를 검토, 감사, 보증, 지원하지 않습니다**. 설치하기 전에 번들 매니페스트, 컴포넌트 카탈로그, 소스 저장소를 검토하고 여러분의 판단에 따라 사용하세요.

번들은 기존 Spec Kit 컴포넌트(확장, 프리셋, 워크플로, 스텝)를 하나의 역할 또는 팀 스택으로 합성합니다. 사용자가 여러 개의 설치 명령을 따로 따르는 대신 테스트를 거친 컴포넌트 집합을 함께 설치할 수 있어야 할 때 유용합니다.

수락된 커뮤니티 번들 항목은 [`bundles/catalog.community.json`](https://github.com/github/spec-kit/blob/main/bundles/catalog.community.json)에 게시되며 아래에 나열됩니다. 내장 커뮤니티 소스는 탐색 전용입니다. `specify bundle search`와 `specify bundle info`로 항목을 살펴볼 수 있지만, ID로 설치하려면 검증된 설치 허용 카탈로그를 명시적으로 추가해야 합니다. 명시적 카탈로그는 내장 커뮤니티 소스보다 기본 우선순위가 높습니다. 메타데이터 검증을 위해 번들을 제출하려면 [게시 가이드](../bundles/publishing.md)를 따르고 [Bundle Submission](https://github.com/github/spec-kit/issues/new?template=bundle_submission.yml) 이슈를 제출하세요.

| 번들 | 용도 | 역할 또는 팀 | 제공 | 필요한 카탈로그 | URL |
|--------|---------|--------------|----------|-------------------|-----|
| Agentstandards Multi-Vendor Architecture | Codex가 오케스트레이션하는 멀티 벤더 아키텍처 위원회, 엄격한 태스크 게이트, 이를 지원하는 Spec Kit 워크플로를 설치합니다. | `architecture-governance` | 확장 1개, 프리셋 1개, 워크플로 1개 | 문서화됨 | [agentstandards](https://github.com/bbjwz/agentstandards) |
| SicarioSpec Security & Governance Bundle | GitHub Spec Kit을 위한 기본 보안 거버넌스 번들. 데이터 분류, 위협 모델링, 코드 소유 검증 게이트를 강제합니다. | `security-engineer` | 확장 1개, 프리셋 11개 | 문서화됨 | [sicario-spec](https://github.com/dfirs1car1o/sicario-spec) |
| Spec Kit for Azure DevOps | ado 확장과 ado-lifecycle 워크플로: 모든 보드 또는 PR 변경에 사람의 게이트를 두고 ADO 스토리를 Done까지 진행합니다. | `developer` | 확장 1개, 워크플로 1개 | 문서화됨 | [spec-kit-ado](https://github.com/praveenlavu/spec-kit-ado) |
| SpecAssay | 표준 Spec Kit을 위한 지속 ID 승격: 템플릿, Gate 2 거부, 추적 매니페스트 출력. | `developer` | 확장 1개, 프리셋 1개 | 문서화됨 | [specassay](https://github.com/rdryfoos/specassay) |

## 함께 보기

- [번들 개요](../bundles/overview.md)
- [번들 설치 및 조회](../bundles/cli.md)
- [카탈로그 정책과 컴포넌트 소스 검증](../bundles/catalogs.md)
- [번들 개발하기](../bundles/developing.md)
- [게시, 업데이트, 제출 검토 범위](../bundles/publishing.md)
