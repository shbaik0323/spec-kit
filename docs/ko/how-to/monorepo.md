# 모노레포에서 Spec Kit 사용하기

Spec Kit 프로젝트는 **디렉터리 범위**입니다. `.specify/`를 포함하는 디렉터리가
곧 프로젝트입니다. 모노레포는 하나의 저장소 루트 아래에 여러 독립적인 Spec Kit
프로젝트를 둘 수 있으며, 각각 자체 `.specify/`, `specs/`, 컨스티튜션, 기능 번호
체계를 갖습니다.

루트 해석은 이미 Git 최상위보다 **가장 가까운** `.specify/`를 우선하므로, 멤버
프로젝트 안에서 실행한 명령은 저장소 루트가 아니라 그 프로젝트로 해석됩니다.

저장소 구조는 인터페이스 설계와 별개입니다. 프로젝트들이 상호 작용할 때는
[계약 주도 개발](contract-driven-development.md)을 사용해 계약마다 권위 있는
소유자를 하나 두고 그들 사이의 합의를 정의하세요. 같은 저장소의 프로젝트는 그
계약을 직접 참조할 수 있고, 별도 저장소의 프로젝트는 버전이 지정된 아티팩트나
동기화된 사본을 사용할 수 있습니다.

## 구조

```text
my-monorepo/
├── .git/                     # one Git repository at the root
├── apps/
│   ├── web/
│   │   └── .specify/         # Spec Kit project "web"
│   │       └── memory/constitution.md
│   └── api/
│       └── .specify/         # Spec Kit project "api"
│           └── memory/constitution.md
└── packages/
    └── ui/
        └── .specify/         # Spec Kit project "ui"
```

각 멤버 프로젝트를 독립적으로 초기화합니다.

```bash
specify init apps/web --integration claude
specify init apps/api --integration claude
```

각 프로젝트는 자체 `specs/` 디렉터리를 유지하고 기능 번호를 독립적으로
매깁니다(`apps/web/specs/001-…`, `apps/api/specs/001-…`).

## 멤버 프로젝트 안에서 작업하기

기본 워크플로는 달라지지 않습니다. 프로젝트 디렉터리로 이동해 슬래시 명령을
실행하세요. 루트 해석이 가장 가까운 `.specify/`를 찾습니다.

```bash
cd apps/web
# 그런 다음 에이전트에서 /speckit.specify, /speckit.plan, … 실행
```

## 저장소 루트에서 멤버 프로젝트 대상으로 지정하기

`cd`를 하고 싶지 않은 비대화형 또는 CI 실행에서는 **`SPECIFY_INIT_DIR`**을 멤버
프로젝트 루트(`.specify/`를 *포함하는* 디렉터리)로 설정하세요. 상대 경로는 현재
디렉터리를 기준으로 해석됩니다.

```bash
# 모노레포 루트에서 apps/web을 대상으로 작업(cd 불필요)
export SPECIFY_INIT_DIR=apps/web
```

경로가 존재하고 `.specify/`를 포함해야 합니다. 그렇지 않으면 명령은 **오류를 내며
현재 디렉터리나 Git 최상위로 폴백하지 않습니다**. 이는 의도된 것입니다. 오타
때문에 엉뚱한 프로젝트에 스펙이 기록되는 일이 없습니다. 존재하지 않는 경로는
입력한 그대로 보고되고, 존재하지만 Spec Kit 프로젝트가 아닌 경로는 해석된 절대
경로로 보고됩니다.

```text
# SPECIFY_INIT_DIR=apps/wbe  (typo: no such directory)
ERROR: SPECIFY_INIT_DIR does not point to an existing directory: apps/wbe

# SPECIFY_INIT_DIR=apps  (exists, but has no .specify/ of its own)
ERROR: SPECIFY_INIT_DIR is not a Spec Kit project (no .specify/ directory): /home/you/my-monorepo/apps
```

`SPECIFY_INIT_DIR`은 **프로젝트**를 선택하고, `SPECIFY_FEATURE_DIRECTORY`는
**기능**을 선택합니다. 상대 기능 경로는 포함 여부 검사 없이 그 프로젝트 루트에
결합되므로 `../shared-feature` 같은 값은 여전히 루트 밖으로 해석됩니다. 절대
경로는 그대로 사용되며 마찬가지로 프로젝트 루트 밖을 가리킬 수 있습니다. 어느
쪽이든 선택된 기능 디렉터리가 그 안에 있어야 하는 것은 아닙니다. 프로젝트 루트는
Spec Kit 프로젝트와 그 명령/설정 컨텍스트를 식별할 뿐, 기능 문서의 포함 경계가
아닙니다. 둘은 함께 쓸 수 있습니다. 둘 다 설정해 프로젝트와 기능을 비대화형으로
선택하세요. 전체 계약과 두 축 모델은
[`SPECIFY_INIT_DIR` 레퍼런스](../overview/core.md#environment-variables)를
참고하세요.

`specify` CLI의 프로젝트 범위 하위 명령도 같은 변수를 따르므로, 마찬가지로 `cd`
없이 루트에서 멤버 프로젝트를 대상으로 삼습니다.

```bash
export SPECIFY_INIT_DIR=apps/web
specify workflow list          # apps/web의 워크플로 나열
specify integration status     # apps/web의 통합 보고
```

검증 규칙은 같습니다. 경로가 존재하고 `.specify/`를 포함해야 하며, 현재
디렉터리로의 폴백은 없습니다.

## `SPECIFY_INIT_DIR`이 에이전트에 전달되는 방식

`SPECIFY_INIT_DIR`은 슬래시 명령이 호출하는 셸 스크립트(Bash의 `get_repo_root`,
PowerShell의 `Get-RepoRoot`)가 읽습니다. 그 스크립트를 실행하는 셸의 환경에 이
변수가 있을 때만 효력이 있습니다.

- **스크립트 / CI 실행:** 명령을 구동하는 같은 셸에서 export하세요. 그곳에서는
  안정적으로 동작합니다.
- **대화형 에이전트:** export한 변수가 에이전트가 사용하는 셸 도구에 전달되는지는
  에이전트마다 다릅니다. 에이전트를 실행하기 *전에* `SPECIFY_INIT_DIR`을
  export하고 한 번 확인하세요(예: `/speckit.specify`를 실행하고 새 기능이 의도한
  프로젝트의 `specs/` 아래에 생겼는지 확인).

## 모노레포에서의 Git

> [!NOTE]
> Spec Kit 프로젝트 파일은 **해석된 프로젝트 루트**로 범위가 한정되지만, Git
> 작업은 여전히 그것을 포함하는 Git 작업 트리에서 실행됩니다. 루트에 Git 저장소가
> 하나 있고 프로젝트가 하위 디렉터리에 있는 모노레포에서는 기능 브랜치 생성이
> 공유된 루트 저장소에서 브랜치를 만들거나 전환합니다. 스펙 디렉터리는 여전히
> 선택된 멤버 프로젝트 아래에 있지만, Git 브랜치 네임스페이스는 모노레포 전체가
> 공유합니다. 저장소 루트에서 브랜치와 커밋을 관리하거나, 프로젝트별로 격리된
> 브랜치 네임스페이스를 원한다면 멤버 프로젝트마다 Git을 초기화하세요.

## 컨스티튜션

각 멤버 프로젝트에는 자체 `.specify/memory/constitution.md`가 있고
`/speckit.constitution`은 로컬 프로젝트의 파일을 편집합니다. Spec Kit은 내장된
기반/상속 메커니즘을 제공하지 않습니다. 한 컨스티튜션이 모노레포의 다른 곳에 있는
공유 규칙을 참조하게 하고 싶다면 그 연결을 직접 유지 관리해야 합니다. 그렇지
않으면 공유 엔지니어링 규칙을 프로젝트마다 복제하거나 동기화하세요.
