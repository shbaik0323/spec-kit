# Agentic SDD

`/speckit.*` 슬래시 명령은 핵심 스펙 주도 개발(SDD) 프로세스, 즉 코딩 에이전트가 단계별로 실행하는 **에이전트 기반 프로세스**를 이끕니다. 처음부터 끝까지 안내를 따라 실행해 보려면 [빠른 시작 가이드](../quickstart.md)를 참고하세요. 이 페이지는 각 명령의 인자, 출력, 상호 작용 방식을 포함한 상세 레퍼런스입니다. 프로세스의 바탕이 되는 철학은 [스펙 주도 개발 이해하기](../how-to/sdd.md)를, 버그 분류는 [Agentic Bug Fix](agentic-bugfix.md)를 참고하세요.

명령은 순서대로 실행하도록 설계되었지만, `/speckit.plan` 전에 엄격하게 필요한 것은 `/speckit.specify`뿐입니다. clarify, checklist, analyze 명령은 의미 있는 모호함이 있는 작업에 추가하는 품질 게이트입니다.

명세에 착수하기 전에 아이디어를 평가하려면
[Agentic Idea Assessment](agentic-assessment.md)를 참고하세요. 평가는 선택
사항이며 SDD 프로세스와 독립적입니다.

> [!NOTE]
> 이 페이지 전체에서 명령은 `/speckit.*` 형식으로 표기합니다. GitHub Copilot의 기본 스킬 모드는 `/speckit-*`을 사용하고, 일부 다른 에이전트는 `$speckit-*`(예: Codex, ZCode)이나 `/skill:speckit-*`(예: Kimi)을 사용합니다. [명령 호출](../integrations/cli.md#command-invocation)을 참고하세요.

```text
/speckit.constitution -> /speckit.specify -> /speckit.clarify -> /speckit.plan -> /speckit.checklist -> /speckit.tasks -> /speckit.analyze -> /speckit.implement -> /speckit.converge
```

## 명령 개요

`specify init` 이후 선택한 통합에서 다음 명령을 사용할 수 있습니다. 선택적 스킬
모드를 지원하는 통합은 `--integration-options="--skills"`로 이를 선택할 수
있으며, Copilot을 포함한 일부 통합은 기본적으로 스킬을 사용합니다.

| 명령 | 에이전트 스킬 | 용도 |
| --- | --- | --- |
| `/speckit.constitution` | `speckit-constitution` | 프로젝트 원칙 수립 또는 갱신 |
| `/speckit.specify` | `speckit-specify` | 요구사항과 사용자 스토리 정의 |
| `/speckit.plan` | `speckit-plan` | 기술 구현 계획 작성 |
| `/speckit.tasks` | `speckit-tasks` | 계획을 실행 가능한 태스크로 분해 |
| `/speckit.implement` | `speckit-implement` | 태스크 실행 |
| `/speckit.converge` | `speckit-converge` | 아티팩트에 비추어 구현을 평가하고 남은 작업 추가 |
| `/speckit.clarify` | `speckit-clarify` | 계획 전에 모호함 해소(선택적 품질 게이트. 이전 이름 `/quizme`) |
| `/speckit.analyze` | `speckit-analyze` | 태스크 이후 구현 전에 아티팩트 일관성 확인(선택적 품질 게이트) |
| `/speckit.checklist` | `speckit-checklist` | 요구사항 품질 체크리스트 생성(선택적 품질 게이트) |

## `/speckit.constitution`

프로젝트 **컨스티튜션**, 즉 이후 모든 단계의 평가 기준이 되는 지도 원칙을 만들거나 갱신하고, 의존하는 템플릿을 동기화된 상태로 유지합니다. 처음에 한 번 실행하고 원칙이 바뀔 때마다 갱신하세요. 원칙을 인자로 전달합니다.

```text
/speckit.constitution This project follows a "Library-First" approach. All features must be implemented as standalone libraries first. We use TDD strictly. We prefer functional programming patterns.
```

더 일반적인 출발점은 다음과 같습니다.

```text
/speckit.constitution Create principles focused on code quality, testing standards, user experience consistency, and performance requirements.
```

## `/speckit.specify`

자연어 설명으로부터 기능 **명세**를 만들거나 갱신합니다. 기술 스택이 아니라 **무엇을**, **왜**, 즉 사용자에게 보이는 동작과 목표에 집중하세요. 기술 스택은 `/speckit.plan`에 속합니다.

이 워크플로는 `/speckit.specify`가 만들고 `/speckit.clarify`가 재평가하는 내장 스펙 품질 체크리스트인 `checklists/requirements.md`도 유지 관리할 수 있습니다. 이 라이프사이클은 `/speckit.checklist`가 생성하는 맞춤 체크리스트와는 별개입니다.

```text
/speckit.specify Build an application that can help me organize my photos in separate photo albums. Albums are grouped by date and can be re-organized by dragging and dropping on the main page. Albums are never in other nested albums. Within each album, photos are previewed in a tile-like interface.
```

## `/speckit.clarify`

현재 스펙에서 충분히 명시되지 않은 영역에 대해 초점을 맞춘 질문을 최대 다섯 개 하고, 답변을 `spec.md`에 다시 반영합니다. 계획 전에 필요한 만큼 여러 번 실행하되, 매번 다른 영역을 다루세요. 선택적으로 집중할 영역을 인자로 전달할 수 있습니다.

```text
/speckit.clarify Focus on the task card behavior: status changes, comment limits, and who can be assigned.
```

계획 전에 명확히 해 두면 모호함 위에 설계하는 일을 막을 수 있습니다. 나중에 `/speckit.analyze`가 요구사항 누락을 드러내면 돌아와서 `/speckit.clarify`(또는 `/speckit.specify`)를 다시 실행하세요.

`checklists/requirements.md`가 있으면 `/speckit.clarify`는 스펙을 다듬는 과정에서 그 평가 상태를 갱신할 수 있습니다. 이 예외는 내장 요구사항 체크리스트에만 적용되며 맞춤 검토 체크리스트에는 적용되지 않습니다.

## `/speckit.plan`

계획 프로세스를 실행해 스펙으로부터 설계 아티팩트를 생성합니다. 구현 세부 사항은 여기에 속합니다. 기술 스택, 아키텍처, 기술적 제약을 인자로 제공하세요.

```text
/speckit.plan Use .NET Aspire with Postgres. The frontend is Blazor Server with drag-and-drop boards and real-time updates. Expose REST APIs for projects, tasks, and notifications.
```

위의 사진 앨범 예시라면 다음과 같습니다.

```text
/speckit.plan The application uses Vite with minimal number of libraries. Use vanilla HTML, CSS, and JavaScript as much as possible. Images are not uploaded anywhere and metadata is stored in a local SQLite database.
```

## `/speckit.checklist`

기능에 대한 품질 체크리스트를 생성합니다. **"요구사항을 위한 단위 테스트"**라고 생각하세요. 코드를 테스트하는 대신 스펙 자체가 완전하고, 명확하고, 모호하지 않고, 일관적인지 확인합니다(예: "모든 열에 대해 드래그 앤 드롭 규칙이 정의되어 있는가?", "담당자로 지정된 사용자가 삭제되었을 때의 동작이 명시되어 있는가?").

이 명령이 생성하는 맞춤 체크리스트는 검토자가 소유하는 요구사항 품질 검토 아티팩트입니다. 명시적으로 요청받으면 에이전트가 평가를 도울 수 있지만, 구현 과정에서 말없이 스스로 승인해서는 안 됩니다. 맞춤 체크리스트에서 `[x]`는 검토자가 요구사항 품질 기준이 충족되었다고 판단했다는 뜻이며, 구현 작업이 완료되었다는 뜻이 아닙니다.

폭넓게 살펴보려면 인자 없이 실행하고, 한 측면을 겨냥하려면 집중할 영역을 전달하세요.

```text
/speckit.checklist
```

```text
/speckit.checklist Focus on the Kanban board interactions and comment permissions.
```

생성된 체크리스트를 검토하세요. 누락이 드러나면 작업을 분해하기 전에 `/speckit.clarify`나 `/speckit.specify`로 돌아가 스펙을 다듬은 다음, 요구사항 품질 기준이 검토되고 충족된 후에만 각 맞춤 체크리스트 항목을 `[x]`로 표시하세요.

## `/speckit.tasks`

설계 아티팩트로부터 의존성 순서로 정렬된 실행 가능한 `tasks.md`를 생성합니다. 태스크는 단계(phase)로 구성됩니다. **Setup**, **Foundational**(선행되어야 하는 차단 요소), 이어서 우선순위 순으로 **사용자 스토리마다 한 단계**, 그리고 공통 관심사를 위한 마지막 **Polish** 단계입니다. 테스트는 요청 시 별도의 단계가 아니라 사용자 스토리의 단계 안에 생성되며, 가능한 경우 태스크에 병렬 실행 표시가 붙습니다.

```text
/speckit.tasks
```

## `/speckit.analyze`

`spec.md`, `plan.md`, `tasks.md` 전반에 걸쳐 **읽기 전용**으로 아티팩트 간 일관성 및 품질 분석을 수행하고 충돌, 누락, 모호함을 보고합니다(예: 대응하는 요구사항이 없는 태스크, 스펙과 모순되는 계획상의 선택). 파일을 절대 편집하지 않습니다. 보고서를 만들고, 선택적으로 여러분이 승인할 교정안을 제안할 수 있습니다.

```text
/speckit.analyze
```

아티팩트를 아직 적은 비용으로 조정할 수 있는 구현 전에 실행하세요. 문제가 드러나면 **그 문제를 소유한 앞 단계로 돌아가** 원본에서 고치세요. 요구사항 문제는 `/speckit.specify`나 `/speckit.clarify`, 설계 문제는 `/speckit.plan`, 태스크 목록 재생성은 `/speckit.tasks`입니다. 그런 다음 깨끗한 결과가 나올 때까지 `/speckit.analyze`를 다시 실행하세요. 추가 검토 삼아 구현 후에 `/speckit.analyze`를 다시 실행할 수도 있습니다.

## `/speckit.implement`

`tasks.md`의 태스크를 실행하며, 각 단계를 의존성 순서대로 실행하고 병렬 표시를 존중합니다.

태스크를 실행하기 전에 체크리스트의 체크박스 상태를 게이트로 읽습니다. 이 명령에게 체크리스트 표시는 읽기 전용입니다. `/speckit.implement`는 체크된 항목과 체크되지 않은 항목을 세고 체크되지 않은 항목이 있으면 진행하기 전에 묻지만, 체크리스트 표시를 변경해서는 안 됩니다. 맞춤 체크리스트에서 체크된 항목은 요구사항 품질에 대한 검토자의 승인을 뜻하며, 완료된 구현 작업을 뜻하지 않습니다.

작은 기능이라면 한 번 실행해 전부 빌드합니다.

```text
/speckit.implement
```

큰 기능이라면 에이전트의 컨텍스트에 과부하가 걸리지 않도록 단계적으로 작업하세요. 인자로 각 실행의 범위를 정하고, 결과를 검증한 다음 계속합니다.

```text
/speckit.implement Implement only the Setup and Foundational phases: project scaffolding and the project/task data model with basic CRUD. Stop before the user-story features.
```

```text
/speckit.implement Now implement the Kanban board user story: drag-and-drop between columns.
```

다음으로 넘어가기 전에 각 단계가 동작하는지 확인하세요.

## `/speckit.converge`

코드베이스를 기능의 스펙, 계획, 태스크에 비추어 평가해 빠진 것이 없는지 확인합니다. **추가 전용(append-only)**입니다. 코드를 절대 편집하거나 삭제하지 않으며, 가능한 유일한 쓰기는 `tasks.md`에 태스크를 추가하는 것입니다. 현재 `tasks.md`에 대해 `/speckit.implement`가 실행된 후에만 실행하세요.

```text
/speckit.converge
```

먼저 심각도 등급이 매겨진 발견 사항 요약을 출력한 다음, 두 가지 결과 중 하나로 귀결됩니다.

- **Converged** — 누락이 발견되지 않았습니다. `tasks.md`는 바이트 단위로 변경 없이 유지되고 `✅ Converged — the implementation satisfies the spec, plan, and tasks.` 같은 깨끗한 결과가 표시됩니다. 끝났습니다. 리뷰로 넘어가거나 PR을 여세요.
- **Tasks appended** — 누락이 발견되었습니다. Converge는 이를 `tasks.md`의 Convergence 섹션 아래에 새 태스크로 추가하고 몇 개인지 알려 줍니다. `/speckit.implement`를 다시 실행해 완료한 다음 `/speckit.converge`를 한 번 더 실행하세요. 패스마다 발견 항목이 줄어듭니다. converged가 보고될 때까지 반복하세요.
