# Agentic Bug Fix

**bug** 확장은 평가, 수정, 검증의 세 단계 버그 분류 프로세스를 추가하며, 코딩 에이전트가 핵심 [Agentic SDD](agentic-sdd.md) 프로세스와 나란히 실행합니다. 각 버그는 `.specify/bugs/<slug>/` 아래의 자체 디렉터리에 있고, 단계마다 Markdown 보고서 하나를 갖습니다.

안내가 있는 예시는 [버그 수정 빠른 시작](../guides/bugfix.md)을 따르세요.
버그 수정에는 SDD 기능 프로세스가 선행될 필요가 없습니다. 깨진 동작을 수리하는
것이 아니라 아이디어를 평가하려면
[아이디어 평가](agentic-assessment.md)를 참고하세요.

> [!NOTE]
> 이 페이지 전체에서 명령은 `/speckit.bug.*` 형식으로 표기합니다. GitHub Copilot의 기본 스킬 모드는 `/speckit-bug-*`를 사용하고, 일부 다른 에이전트는 `$speckit-bug-*`(예: Codex, ZCode)이나 `/skill:speckit-bug-*`(예: Kimi)을 사용합니다. [명령 호출](../integrations/cli.md#command-invocation)을 참고하세요.

bug 확장은 번들로 제공되는 선택형 확장입니다. 이 명령들을 사용하기 전에 설치하세요.

```bash
specify extension add bug
```

세 명령은 하나의 핸들, 즉 `.specify/bugs/` 아래의 버그별 디렉터리 이름인 **slug**를 공유합니다. `slug=<name>`으로 제공하세요. 생략하면 `/speckit.bug.assess`가 묻습니다(자동화 모드에서는 고유한 slug를 생성). slug는 소문자 케밥 케이스로 정규화됩니다. 해당 slug에 대한 평가가 이미 있으면 대화형 실행은 덮어쓰기 전에 묻고, 자동화된 실행은 거부한 뒤 새로운 고유 slug를 고릅니다.

```text
/speckit.bug.assess -> /speckit.bug.fix -> /speckit.bug.test
```

## `/speckit.bug.assess`

붙여 넣은 텍스트(스택 트레이스 등)나 URL(GitHub 이슈 등) 형태의 버그 보고를 코드베이스에 비추어 분류합니다. 보고가 실제 버그인지 판단하고, 의심되는 코드 경로를 찾고, 교정 방안을 제안합니다. 이 명령은 **읽기 전용**입니다. `assessment.md`만 작성하며 소스 코드를 절대 수정하지 않습니다.

```text
/speckit.bug.assess "TypeError: cannot read properties of undefined (reading 'token') at /auth/callback"
```

```text
/speckit.bug.assess https://github.com/example/repo/issues/1234 slug=callback-token
```

출력: `.specify/bugs/<slug>/assessment.md`.

## `/speckit.bug.fix`

평가에 기술된 교정 방안을 적용하고 정확히 무엇이 바뀌었는지 기록합니다. 소스 코드를 편집하는 bug 명령은 이것**뿐**이며, 새로운 근거로 인해 범위를 넓혀야 하는 경우(**Deviations from Assessment** 아래에 기록)가 아니면 평가에 나열된 파일 안에 머뭅니다.

```text
/speckit.bug.fix slug=callback-token
```

출력: `.specify/bugs/<slug>/fix.md`.

## `/speckit.bug.test`

재현 절차와 추가된 테스트를 다시 실행해 수정을 검증한 다음 검증 결과를 `verified`, `partial`, `failed` 중 하나로 기록합니다. `assess`와 마찬가지로 소스 코드에 대해서는 **읽기 전용**입니다. 판정은 절대 과장되지 않습니다. 평가에 나열된 재현 절차가 실제로 수행되지 않았다면 전체 결과는 `verified`로 보고되지 않고 `partial`로 낮아집니다.

```text
/speckit.bug.test slug=callback-token
```

출력: `.specify/bugs/<slug>/test.md`.

## 확장 훅

각 bug 명령은 다른 확장을 위해 `before_bug_<command>`와 `after_bug_<command>` 훅을 노출합니다. 사전 훅은 slug 해석과 선행 조건 확인 후에 실행됩니다. 평가는 버그 디렉터리를 먼저 만듭니다. 사후 훅은 보고서가 작성된 후, 완료가 보고되기 전에 실행됩니다. 알아볼 수 없는 붙여 넣은 보고는 평가를 작성하거나 사후 훅을 실행하지 않고 중단됩니다. 사전 훅은 이미 실행되었을 수 있습니다. 훅은 같은 세션에서 실행되는 프롬프트입니다. 대화에서 해석된 `BUG_SLUG`와 `BUG_DIR`(그리고 사후 훅의 경우 작성된 보고서)을 재사용하며, 변수는 주입되지 않습니다. 등록된 훅이 없으면 워크플로는 이전과 같이 진행됩니다. [bug 확장의 훅 이벤트와 설정 예시](https://github.com/github/spec-kit/blob/main/extensions/bug/README.md#hooks)를 참고하세요.
