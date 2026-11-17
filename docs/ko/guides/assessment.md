# 아이디어 평가 빠른 시작

해결책에 착수하기 전에 아이디어에 투자할 가치가 있는지 결정할 때 이 프로세스를
사용하세요. 소프트웨어 아이디어와 소프트웨어가 아닌 결정 모두에 쓸 수 있습니다.
기존 소스 코드는 필요하지 않습니다.

번들로 제공되는 선택형 **assess** 확장은 다섯 단계를 제공합니다.
**intake → research → define → shape → decide**. 결과물은 구현이 아니라 근거에
기반한 `go`, `needs-clarification`, `kill` 결정입니다.

## 설정

[Spec Kit을 설치하고 프로젝트를 초기화하세요](../installation.md). 빈 프로젝트로도
충분하며, 아이디어가 코드와 관련 있다면
[기존 저장소](../how-to/existing-projects.md)를 사용할 수도 있습니다.
초기화된 프로젝트 루트의 터미널에서 확장을 설치합니다.

```bash
specify extension add assess
```

그런 다음 그 디렉터리에서 코딩 에이전트를 실행하세요. 아래 예시는 GitHub
Copilot의 기본 스킬 모드(`--integration copilot`)를 사용합니다. 다른 에이전트는
각자의
[명령 호출 구문](../integrations/cli.md#command-invocation)을 사용합니다.
각 `/speckit-assess-*` 스킬을 에이전트의 채팅에서 따로따로 호출하고, 다음으로
넘어가기 전에 아티팩트를 검토하세요. 이들은 터미널 명령이 아니라 에이전트
스킬입니다. 위의 터미널 명령은 확장을 설치할 뿐입니다.

## 1. 아이디어 포착

```text
/speckit-assess-intake "Let users work offline and sync when they reconnect." slug=offline-mode
```

URL, 티켓, 코드베이스 포인터를 제공할 수도 있습니다. 에이전트는 입력을
`.specify/assessments/offline-mode/intake.md`에 담습니다.
slug는 다섯 단계 모두가 공유하는 핸들입니다.

## 2. 근거 조사

```text
/speckit-assess-research slug=offline-mode
```

에이전트는 찬성 및 반대 근거를 `research.md`에 모읍니다. 출처와 신뢰 수준을
검토하세요. 인용 없는 가정은 확립된 사실이 아닙니다. 이 단계는 아이디어를 옹호하는
논거만 쌓는 것이 아니라 아이디어에 이의를 제기해야 합니다.

## 3. 문제 정의

```text
/speckit-assess-define slug=offline-mode
```

에이전트는 영향받는 사용자, 문제, 목표, 비목표, 성공 지표, 아무것도 하지 않을 때의
비용을 설명하는 `problem.md`를 작성합니다.

## 4. 가능한 해결책 구체화

```text
/speckit-assess-shape slug=offline-mode
```

에이전트는 개념 수준의 선택지, 제약, 투입 의향(appetite), 절충점을 `concept.md`에서
비교합니다. 상세 아키텍처, 구현 태스크, 소스 변경은 의도적으로 이 프로세스의 범위
밖입니다.

## 5. 결정 내리기

```text
/speckit-assess-decide slug=offline-mode
```

에이전트는 스코어카드, 근거, 판정을 `decision.md`에 기록합니다.

| 판정 | 의미 | 할 일 |
| --- | --- | --- |
| `go` | 문제와 근거가 충분하고 권장되는 개념이 있음 | 실행 여부를 결정. 소프트웨어 아이디어라면 선택적으로 SDD에 인계 |
| `needs-clarification` | 특정 미지수가 여전히 방어 가능한 결정을 가로막음 | 누락된 정보로 영향받는 아티팩트를 다듬기 |
| `kill` | 지금은 추진할 가치가 없는 아이디어 | 기록된 근거를 보관. 중단은 성공적인 결과입니다 |

다섯 아티팩트 모두 `.specify/assessments/offline-mode/` 아래에 있습니다. 평가
명령은 어느 것도 소스 코드를 수정하지 않습니다.

<a id="resolve-unknowns-by-refining-the-artifacts"></a>

## 아티팩트를 다듬어 미지수 해소하기

일반적인 흐름에서는 각 명령을 한 번씩 실행합니다. 아티팩트에
`[NEEDS CLARIFICATION: ...]`이 있으면 그 Markdown 파일을 직접 편집하거나, 자유
형식 채팅으로 에이전트에게 누락된 정보를 반영해 달라고 요청하세요.

```text
Update .specify/assessments/offline-mode/problem.md with these confirmed success metrics: <your metrics>. Update any dependent wording and revise decision.md if this resolves the blocker. Preserve source and confidence tags.
```

단계 전체를 자동으로 재생성하지 마세요. 새로운 사실이 차단 요인을 해소하는지
검토하고 그에 따라 후속 스코어카드, 판정, 인계 내용을 수정해 달라고 에이전트에게
요청하세요. 명령을 다시 실행하는 것은 폐기한 초안을 교체하는 경우 같은 예외이며,
기존 아티팩트는 확인 없이 덮어써서는 안 됩니다.

## SDD로의 선택적 인계

평가는 독립적입니다. SDD를 자동으로 호출하거나 명세 프로세스에 끼어들지
않습니다. `go` 판정을 받은 소프트웨어 아이디어를 만들기로 했다면, 필요한 경우
프로젝트 컨스티튜션을 수립한 다음 결정 문서의 인계 요약을 명세 명령에 전달하세요.

```text
/speckit-specify Use the handoff summary in .specify/assessments/offline-mode/decision.md to specify the approved offline-mode concept.
```

[SDD 빠른 시작](../quickstart.md)으로 이어 가세요. 단계별 선행 조건, slug 규칙,
근거 요구 사항은
[아이디어 평가 명령 레퍼런스](../reference/agentic-assessment.md)를 참고하세요.
