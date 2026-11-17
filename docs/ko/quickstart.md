<a id="quick-start-guide"></a>

# 스펙 주도 개발 빠른 시작

이 가이드는 Spec Kit으로 스펙 주도 개발(Spec-Driven Development)을 시작하도록 돕습니다. 전체 과정에서 소규모 팀 생산성 플랫폼인 **Taskify**라는 예제로 각 단계를 설명합니다.

SDD는 명세로부터 기능이나 애플리케이션을 만들 때 사용하세요. 수리 작업이라면
[버그 수정 빠른 시작](guides/bugfix.md)을, 아이디어에 투자할 가치가 있는지
결정하려면 [아이디어 평가 빠른 시작](guides/assessment.md)을 따르세요.
이들은 독립적인 프로세스이며 이 가이드의 선행 조건이 아닙니다.

> [!NOTE]
> 자동화 스크립트는 Bash(`.sh`), PowerShell(`.ps1`), Python(`.py`) 변형으로 제공됩니다. 대화형 `specify init`은 하나를 선택하라고 묻고, 비대화형 실행(TTY 없음 또는 `--non-interactive`)은 OS에 맞는 셸 변형을 기본값으로 씁니다. 명시적으로 선택하려면 `--script sh|ps|py`를 전달하세요.

여기의 프로세스 단계는 GitHub Copilot의 기본 스킬 모드(`/speckit-*`)를
사용합니다. 다른 에이전트나 모드는
[명령 호출](integrations/cli.md#command-invocation)을 참고하세요.
각 `/speckit-*` 스킬을 에이전트의 채팅에서 따로따로 호출하고, 다음 단계로
넘어가기 전에 결과를 검토하세요. 이들은 터미널 명령이 아니라 에이전트
스킬입니다. CLI 설치와 프로젝트 설정만 터미널을 사용합니다.

## 권장 프로세스

> [!TIP]
> **컨텍스트 인식**: Spec Kit은 `.specify/feature.json`에 기록된 기능 디렉터리로 활성 기능을 추적합니다(`SPECIFY_FEATURE_DIRECTORY` 환경 변수로 재정의 가능). 명령은 체크아웃된 Git 브랜치가 **아니라** 이 상태에서 기능을 결정하므로 Git이 필요하지 않습니다. 선택형 **git** 확장은 버전 관리에서 작업을 정리할 수 있도록 번호가 붙은 기능 브랜치(예: `001-feature-name`)를 추가하지만, 활성 기능은 여전히 그 상태가 가리키는 디렉터리이며 `git checkout`만으로는 바뀌지 않습니다. 명령이 다른 기능을 가리키게 하려면 `.specify/feature.json`을 갱신하거나 `SPECIFY_FEATURE_DIRECTORY`를 설정하세요.

Spec Kit을 설치하고 나면 아래의 각 스킬이 프로세스의 한 단계가 됩니다. 흔히 쓰는 경로는 두 가지입니다.

어느 경로든 시작하기 전에 `/speckit-constitution`으로 프로젝트당 한 번
컨스티튜션(constitution, 프로젝트 원칙)을 수립하세요.

**짧은 경로** — 작은 기능용:

1. `/speckit-specify`
2. `/speckit-plan`
3. `/speckit-tasks`
4. `/speckit-implement`
5. `/speckit-converge`

**전체 경로** — 프로덕션 기능용. `/speckit-clarify`, `/speckit-checklist`, `/speckit-analyze`를 품질 게이트로 추가합니다.

1. `/speckit-constitution` (프로젝트당 한 번)
2. `/speckit-specify`
3. `/speckit-clarify`
4. `/speckit-plan`
5. `/speckit-checklist`
6. `/speckit-tasks`
7. `/speckit-analyze`
8. `/speckit-implement`
9. `/speckit-converge`

### Specify 설치

**터미널에서** PyPI로부터 CLI를 설치한 다음([uv](install/uv.md) 필요) 프로젝트를 초기화합니다.

```bash
uv tool install specify-cli
specify init taskify --integration copilot
cd taskify
```

`init`에서는 코딩 에이전트를 대화형으로 고르거나 `--integration`으로 명시적으로 전달할 수 있습니다(예: `--integration copilot`). CI와 AI 에이전트 하네스에서는 `--non-interactive`를 추가해, 지정하지 않은 선택 항목이 화살표 키 선택기에서 멈추지 않고 문서화된 기본값을 사용하도록 하세요.

> [!NOTE]
> `pipx`, 일회성 `uvx` 실행, 고정된 릴리스, 오프라인/에어갭 구성을 선호하시나요? 지원되는 모든 방법은 [설치 가이드](installation.md)를 참고하세요.
> 이미 코드가 있는 저장소에 Spec Kit을 추가하나요? 아래 워크플로를 시작하기
> 전에 [기존 프로젝트에 Spec Kit 도입하기](how-to/existing-projects.md)를
> 따르세요.

프로젝트 디렉터리에서 코딩 에이전트를 실행하세요. 다음 스킬들을 채팅에서
한 번에 하나씩 호출합니다.

<a id="step-1-speckitconstitution--set-the-ground-rules"></a>

### 1단계: `/speckit-constitution` — 기본 규칙 정하기

이후 모든 단계의 평가 기준이 되는 프로젝트의 지도 원칙을 수립합니다. 처음에 한 번, 원칙을 인자로 전달해 실행하세요.

```text
/speckit-constitution Taskify is a "Security-First" application. All user inputs must be validated. We use a microservices architecture. Code must be fully documented.
```

<a id="step-2-speckitspecify--describe-what-to-build"></a>

### 2단계: `/speckit-specify` — 무엇을 만들지 설명하기

자연어 설명으로부터 기능 명세를 만듭니다. 기술 스택이 아니라 **무엇을**, **왜** 만드는지에 집중하세요.

```text
/speckit-specify Develop Taskify, a team productivity platform where predefined users create projects, assign tasks, comment, and move tasks across Kanban columns (To Do, In Progress, In Review, Done). Five users (one product manager, four engineers), three sample projects, no login for this first phase.
```

<a id="step-3-speckitclarify--resolve-ambiguities"></a>

### 3단계: `/speckit-clarify` — 모호함 해소하기

충분히 명시되지 않은 부분에 대해 초점을 맞춘 질문을 하고 답변을 스펙에 반영해, 모호함 위에 계획을 세우지 않도록 합니다. 계획 전에 실행하며, 선택적으로 집중할 영역을 지정할 수 있습니다.

```text
/speckit-clarify Focus on task card behavior — status changes, comment permissions, and user assignment.
```

<a id="step-4-speckitplan--choose-the-tech-stack"></a>

### 4단계: `/speckit-plan` — 기술 스택 선택하기

스펙으로부터 설계 아티팩트를 생성합니다. 구현 세부 사항은 여기에 속합니다. 기술 스택과 아키텍처를 제공하세요.

```text
/speckit-plan Use .NET Aspire with Postgres. The frontend is Blazor Server with drag-and-drop boards and real-time updates. Expose REST APIs for projects, tasks, and notifications.
```

<a id="step-5-speckitchecklist--validate-the-spec"></a>

### 5단계: `/speckit-checklist` — 스펙 검증하기

작업을 분해하기 전에 스펙이 완전하고 명확하며 일관적인지 확인하는 맞춤 품질 체크리스트, 즉 "요구사항을 위한 단위 테스트"를 생성합니다. 이 맞춤 체크리스트는 검토자가 소유하는 요구사항 품질 검토 아티팩트입니다. 검토자가 해당 요구사항 품질 기준이 충족되었다고 판단할 때만 항목을 `[x]`로 표시하세요. 맞춤 항목이 체크되었다고 해서 구현 작업이 완료되었다는 뜻은 아닙니다.

```text
/speckit-checklist
```

<a id="step-6-speckittasks--break-the-work-down"></a>

### 6단계: `/speckit-tasks` — 작업 분해하기

설계 아티팩트로부터 의존성 순서로 정렬된 실행 가능한 `tasks.md`를 생성합니다.

```text
/speckit-tasks
```

<a id="step-7-speckitanalyze--check-consistency"></a>

### 7단계: `/speckit-analyze` — 일관성 확인하기

`spec.md`, `plan.md`, `tasks.md` 전반의 충돌, 누락, 모호함을 보고합니다. 읽기 전용이므로, 문제가 표시되면 원본에서 고치고 구현 전에 다시 실행하세요.

```text
/speckit-analyze
```

<a id="step-8-speckitimplement--build-it"></a>

### 8단계: `/speckit-implement` — 빌드하기

`tasks.md`의 태스크를 의존성 순서대로 실행합니다. 구현 전에 체크리스트의 체크박스 상태를 게이트로 읽고, 체크되지 않은 항목이 있으면 진행하기 전에 묻습니다. 체크리스트 파일이나 표시는 변경하지 않습니다. 내장 `checklists/requirements.md` 체크리스트는 `/speckit-specify`와 `/speckit-clarify`가 유지 관리하고, 맞춤 체크리스트는 검토자 소유로 남습니다. 한 번 실행해 전부 빌드하거나, 큰 기능이라면 한 번에 한 단계(phase)씩 범위를 좁혀 실행하세요.

```text
/speckit-implement
```

<a id="step-9-speckitconverge--verify-completeness"></a>

### 9단계: `/speckit-converge` — 완전성 검증하기

코드베이스를 스펙, 계획, 태스크에 비추어 점검합니다. 누락을 발견하면 `tasks.md`에 새 태스크를 추가합니다. **Converged**가 보고될 때까지 `/speckit-implement`를 실행하고 다시 수렴시키세요. 누락이 없다면 끝입니다. 리뷰로 넘어가거나 PR을 여세요.

```text
/speckit-converge
```

> [!TIP]
> 각 명령의 인자, 출력, 단계별 구현, 상호 작용 방식에 대한 전체 레퍼런스는 [Agentic SDD](reference/agentic-sdd.md)를 참고하세요.

## 핵심 원칙

- 무엇을 왜 만드는지 **명확하게 밝히세요**
- 명세 단계에서는 **기술 스택에 집중하지 마세요**
- 구현 전에 명세를 **반복해서 다듬으세요**
- 코딩을 시작하기 전에 요구사항과 계획을 **검증하세요**
- 구현 세부 사항은 **코딩 에이전트에게 맡기세요**

## 다음 단계

- 모든 명령의 자세한 내용은 [Agentic SDD](reference/agentic-sdd.md) 레퍼런스를 참고하세요
- 확장, 프리셋, 워크플로, 번들로 [프로세스를 사용자 정의하는](how-to/customization.md) 방법을 알아보세요
- 심층적인 안내는 [전체 방법론](https://github.com/github/spec-kit/blob/main/spec-driven.md)을 읽어 보세요
- [핵심 템플릿](https://github.com/github/spec-kit/tree/main/templates)을
  [커뮤니티 워크스루](community/walkthroughs.md)와 비교해 스펙 주도 개발이 실제 프로젝트에서 어떻게 쓰이는지 살펴보세요
- [GitHub의 소스 코드](https://github.com/github/spec-kit)를 둘러보세요
