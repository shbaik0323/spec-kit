# Agentic Idea Assessment

번들로 제공되는 선택형 **assess** 확장은
**intake → research → define → shape → decide**를 통해 아이디어를 평가합니다.
이는 독립적인 프로세스입니다. 소스 코드가 없는 초기화된 프로젝트에서도 실행할 수
있고, 평가 아티팩트만 만들며, SDD를 자동으로 시작하지 않습니다. 안내가 있는
예시는 [아이디어 평가 빠른 시작](../guides/assessment.md)을 참고하세요.

초기화된 프로젝트의 루트에서 설치합니다.

```bash
specify extension add assess
```

아래 명령은 `/speckit.assess.*` 표기를 사용합니다. GitHub Copilot의 기본 스킬
모드는 `/speckit-assess-*`를 사용하며, 다른 통합은 다른 접두사를 사용할 수
있습니다. [명령 호출](../integrations/cli.md#command-invocation)을 참고하세요.

## 명령과 아티팩트

모든 아티팩트는 `.specify/assessments/<slug>/` 아래에 기록됩니다.

| 명령 | 용도 | 출력 |
| --- | --- | --- |
| `/speckit.assess.intake` | 텍스트, URL, 티켓, 코드베이스 포인터로부터 아이디어 포착 | `intake.md` |
| `/speckit.assess.research` | 출처와 신뢰 수준을 갖춘, 아이디어에 대한 찬반 근거 수집 | `research.md` |
| `/speckit.assess.define` | 사용자, 문제, 목표, 비목표, 지표, 방치 비용 정의 | `problem.md` |
| `/speckit.assess.shape` | 개념 수준의 선택지, 투입 의향, 절충점 비교 | `concept.md` |
| `/speckit.assess.decide` | 스코어카드, 판정, 근거, 선택적 SDD 인계 기록 | `decision.md` |

초기 아이디어와 slug를 intake에 전달한 다음 slug를 재사용하세요.

```text
/speckit.assess.intake "Let users work offline and sync when they reconnect." slug=offline-mode
/speckit.assess.research slug=offline-mode
/speckit.assess.define slug=offline-mode
/speckit.assess.shape slug=offline-mode
/speckit.assess.decide slug=offline-mode
```

## 선행 조건과 범위

다섯 단계 순서가 일반적인 경로이지만 모든 단계가 필수는 아닙니다.

- `define`은 사용자 입력만으로 바로 동작할 수 있습니다. 문제 정의를 만드는 데
  intake와 research는 선택 사항입니다.
- `shape`는 `problem.md`가 필요하며 개념 수준에 머뭅니다. 아키텍처, 데이터 모델,
  API, 태스크, 구현은 평가가 아니라 전달에 속합니다.
- `decide`는 `problem.md`가 필요하며 사용 가능한 모든 평가 아티팩트를 읽습니다.
  `go` 판정에는 추가로 구체화된 개념과 충분한 근거가 필요합니다. research를
  건너뛴다고 해서 근거 요구 사항이 면제되지는 않습니다.

## slug와 기존 아티팩트

slug는 아이디어 하나의 디렉터리를 식별합니다. 사용자가 제공한 slug는 소문자
케밥 케이스로 정규화되며 타임스탬프나 숫자가 덧붙지 않고 유지됩니다.
slug가 제공되지 않으면 intake가 대화형으로 묻고, 자동화된 실행은 고유한 slug를
생성합니다. 이후 단계는 해당 디렉터리가 존재하면 컨텍스트의 slug를 재사용할 수
있습니다.

기존 아티팩트는 확인 없이 덮어쓰이지 않습니다. 자동화 모드에서는 명령이 덮어쓰기를
거부합니다. 일상적인 명확화에는 명령을 다시 실행하는 대신 기존 아티팩트를
다듬으세요.

## 결정과 근거

결정 스코어카드는 기준을 `strong`, `adequate`, `weak`, `unknown`으로 평가하며,
아티팩트에서 가져온 근거를 함께 적습니다.

| 판정 | 요구 사항 또는 의미 |
| --- | --- |
| `go` | 문제의 타당성과 근거의 강도가 최소 `adequate`이고, 권장되는 개념 선택지가 있음 |
| `needs-clarification` | 명시된 미지수가 결정을 가로막음. 약하거나 알 수 없는 근거, 또는 누락된 개념으로는 `go`를 정당화할 수 없음 |
| `kill` | 지금은 추진할 가치가 없는 아이디어. 결정적인 이유가 명시적으로 기록됨 |

research에는 아이디어에 반하는 근거가 포함되어야 합니다. 출처 없는 주장은
`ASSUMPTION`으로 표시된 채 남습니다. 인용을 지어내거나 불확실한 근거를 사실처럼
제시하지 마세요. 문서화된 중단 결정은 유용한 결과입니다.

## 평가 다듬기

일반적인 흐름에서는 각 단계를 한 번씩 실행합니다. 명확화 표시나
`needs-clarification` 판정을 해소하려면 영향받는 Markdown을 직접 편집하거나, 자유
형식 채팅으로 에이전트에게 누락된 사실을 반영해 달라고 요청하세요. 출처와 신뢰도
태그를 보존한 다음, 의존하는 아티팩트를 검토하고 필요에 따라 `decision.md`를
수정하세요. 이는 아티팩트 다듬기이지 명령의 반복 실행이 아닙니다.

앞선 명령을 다시 실행하는 것은 잘못된 slug나 폐기한 초안 같은 예외를 위한
것입니다.
[안내가 있는 명확화 예시](../guides/assessment.md#resolve-unknowns-by-refining-the-artifacts)를
참고하세요.

## 인계와 가드레일

`go` 결정에는 `/speckit.specify`에 전달할지 선택할 수 있는 인계 요약이
포함됩니다. 어떤 라이프사이클 훅도 평가를 SDD의 선행 조건으로 설치하지 않으며,
소프트웨어가 아닌 아이디어는 SDD에 아예 들어갈 필요가 없습니다.

평가 명령은 `.specify/assessments/<slug>/` 안에만 기록하며 소스 코드를 편집하지
않습니다. 확장은 slug와 경로를 검사하고, 가져온 콘텐츠를 신뢰할 수 없는 데이터로
취급하고, URL 신뢰 정책을 적용합니다. 전체 가드레일과 다른 확장과의 관계는
[확장 문서](https://github.com/github/spec-kit/blob/main/extensions/assess/README.md)를
참고하세요.
