# 큰 기능을 더 작은 명세로 나누기

기능이 너무 커서 한 번의
`/speckit.specify` → `/speckit.plan` → `/speckit.tasks` → `/speckit.implement`
주기로 처리하면 모델이 구현 도중에 흐름을 놓친다면, 그것을 독립적으로 명세된 더
작은 하위 기능의 **로드맵**으로 나눌 수 있습니다. 이것이 "스펙의 스펙(spec of
specs)" 접근 방식입니다. 사전에 한 번의 패스로 거대한 기능을 자기 완결적인 스펙으로
분해하고, 각 스펙은 자체 specify/plan/tasks/implement 주기를 거칩니다.

> **언제 사용하는가.** 분해는
> [복잡한 기능 다루기](complex-features.md)의 전략 중 오버헤드가 가장 큽니다.
> **그곳의 더 가벼운 선택지로 부족할 때만** 사용하세요. 먼저 `/speckit.implement`
> 호출당 실행하는 태스크 수를 제한해 보고, 다음으로 하위 에이전트 위임을, 그다음
> 둘의 결합을 시도하세요. 단계 하나만으로도 한 번의 실행으로 다루기에 너무 클 때만
> 스펙의 스펙을 쓰세요.

이 페이지의 나머지는 이미 가진 도구로 이를 *어떻게* 하는지 설명합니다. 새 명령이나
확장은 필요하지 않습니다.

## 로드맵 패스

하위 스펙을 작성하기 전에 한 번의 분해 패스로 로드맵을 만드세요. 이를 완전한 스펙이
아니라 에이전트와 나누는 가벼운 계획 대화로 취급하세요.

1. **기능 전체를 진술합니다.** 큰 기능("에픽")을 한두 문장으로 설명해 에이전트가
   처음부터 전체 그림을 갖게 합니다.
2. **독립적인 조각을 식별합니다.** 에이전트에게 각각 에픽의 일관된 한 부분을
   전달하고 단독으로 명세할 수 있는 소수의 하위 기능을 제안하도록 요청합니다.
   독립적으로 테스트할 수 있는 조각을 목표로 하세요. 하나만 구현해도 시연할 수
   있는 무언가가 남아야 합니다.
3. **경계를 긋습니다.** 조각마다 의도 한 줄과 명시적인 범위 경계(무엇이 포함되고,
   무엇이 형제 조각으로 미뤄지는지)를 적습니다. 뚜렷한 경계가 각 하위 스펙을
   컨텍스트에 들어갈 만큼 작게 유지해 줍니다.
4. **의존성 순으로 정렬합니다.** 어느 조각이 다른 조각에 의존하는지 적고 선행
   조건이 먼저 오도록 순서를 정합니다. 서로 의존하지 않는 조각은 어떤 순서로든
   만들 수 있습니다. 독립적인 조각을 병렬로 만들려면 각 실행이 격리된 활성 기능
   상태를 갖도록 별도의 워크트리를 사용하세요.
5. **결과를 로드맵으로 기록합니다.** 이후의 모든 하위 스펙이 다시 가리킬 수 있도록
   조각들을 지속되는 로드맵 파일(아래)에 담습니다.

로드맵은 의도적으로 얕습니다. 하위 기능에 이름을 붙이고 순서를 정하지만 그것을
설계하지는 **않습니다**. 설계는 각 조각이 자체 `/speckit.specify`를 거칠 때
이루어집니다.

## 로드맵 아티팩트

로드맵은 여러분이 작성하고 버전 관리에 두는 일반적인 Markdown 파일입니다. 그 뒤에
특별한 도구는 없습니다. 하위 스펙이 찾을 수 있는 곳에 두세요.

- 기능 범위의 에픽: `specs/<epic-slug>/roadmap.md`.
- 더 크고 여러 영역에 걸친 에픽: 최상위 `ROADMAP.md`.

각 로드맵 항목은 안정적인 id(나중에 연결에 사용), 이름, 의도, 범위 경계, 의존성,
상태, 그리고 하위 스펙이 생기면 그에 대한 링크를 담습니다. 최소한의 템플릿은
다음과 같습니다.

```markdown
# Roadmap: <epic name>

<One or two sentences: what the epic is and why it is being decomposed.>

**Status legend**: planned · in-progress · done

| ID | Sub-feature | Intent | Scope boundary | Depends on | Status | Sub-spec |
|----|-------------|--------|----------------|-----------|--------|----------|
| R1 | <name>      | <one line> | <in / deferred> | —      | planned | — |
| R2 | <name>      | <one line> | <in / deferred> | R1     | planned | — |
| R3 | <name>      | <one line> | <in / deferred> | R1     | planned | — |
```

하위 스펙이 참조하기 시작하면 `ID` 열은 불변으로 유지하세요. 추적성의 기준점이기
때문입니다. 각 하위 기능을 만들 때 그 스펙 디렉터리의 경로로 `Sub-spec` 열을
채우고, 작업이 진행됨에 따라 `Status`를 갱신하세요.

## 각 하위 기능 명세하기

로드맵이 준비되면 평소의 Spec Kit 흐름으로 항목을 하나씩 처리하세요. 새로 배울
것은 없습니다.

1. 의존성이 이미 `done`이거나 의존성이 없는 다음 로드맵 항목을 고릅니다.
2. 그 조각에 대해서만 `/speckit.specify`를 실행하되, 로드맵 항목의 의도와 범위만
   설명합니다. 조각의 범위가 한정되어 있으므로 스펙, 계획, 태스크가 컨텍스트 윈도
   안에 넉넉히 머뭅니다.
3. 그 조각에 대해 평소처럼 `/speckit.plan`, `/speckit.tasks`,
   `/speckit.implement`를 실행합니다.
4. 로드맵 항목을 `done`으로 표시하고 다음으로 넘어갑니다.

각 조각은 자체 `spec.md`/`plan.md`/`tasks.md`를 가진 완전하고 독립적인 Spec Kit
기능입니다. 로드맵이 이들을 하나로 묶습니다.

## 하위 스펙을 로드맵에 연결하기

별도의 실행에 걸쳐 범위와 의도가 흘러가지 않도록, 모든 하위 스펙은 자신의 로드맵
항목을 참조하고 로드맵은 다시 그것을 가리킵니다. 단순하고 grep으로 찾을 수 있는
양방향 관례입니다.

- **하위 스펙 → 로드맵.** 하위 기능의 `spec.md`에서 `Input` / 요약 줄에 상위
  로드맵과 항목 id를 적습니다. 예:

  ```markdown
  **Input**: Parent roadmap: `specs/<epic>/roadmap.md` → entry **R3**. <feature description>
  ```

- **로드맵 → 하위 스펙.** 로드맵 표에서 항목의 `Sub-spec` 열을 하위 기능의
  디렉터리로 설정합니다. 예: `specs/<epic>-part-3/`.

양방향 모두 일반 텍스트이므로, 도구도 메타데이터 스키마도 없이 간단한 검색만으로
어느 하위 스펙이든 에픽 안에서의 위치를 거슬러 추적하고 형제를 찾을 수 있습니다.

## 로드맵과 하위 스펙을 동기화된 상태로 유지하기

로드맵은 살아 있는 문서입니다. 더 많이 알게 되면 로드맵과 하위 스펙을 정렬된
상태로 유지하세요.

- **로드맵 먼저, 그다음 조정.** 범위가 달라지면 먼저 로드맵 항목을 갱신한 다음
  영향받는 하위 스펙을 갱신합니다. 로드맵이 에픽이 어떻게 나뉘는지에 대한 진실
  공급원입니다.
- **의존성과 순서를 존중하세요.** 조각이 다른 조각에 의존한다면 선행 조건을 먼저
  만들고, 관계가 양쪽에서 보이도록 의존하는 하위 스펙에 상호 참조를 답니다.
- **조각이 여전히 너무 크면 재귀하세요.** 하위 기능이 한 주기로 명세하기에 너무
  크다고 드러나면, 그것에 자체 로드맵을 주고 더 분해하세요. 한 단계 아래에서도
  같은 접근 방식이 적용됩니다. 재귀는 오버헤드를 더하므로 컨텍스트 문제가 실제로
  요구하는 깊이까지만 들어가세요.

## 실제 예시

에픽이 **"셀프서비스 결제 포털 추가"**라고 합시다. 한 주기로 처리하기에는 너무
큽니다. 로드맵 패스가 이를 독립적으로 명세할 수 있는 세 조각으로 나눕니다.

`specs/billing-portal/roadmap.md`:

```markdown
# Roadmap: Self-service billing portal

Let customers view invoices, manage payment methods, and change plans without
contacting support. Too large for one cycle, so it is split into independent slices.

**Status legend**: planned · in-progress · done

| ID | Sub-feature        | Intent                                   | Scope boundary                              | Depends on | Status  | Sub-spec |
|----|--------------------|------------------------------------------|---------------------------------------------|-----------|---------|----------|
| R1 | Invoice history    | Customers view and download past invoices | Read-only; no payment actions               | —         | done    | specs/billing-invoices/ |
| R2 | Payment methods    | Add, remove, and set a default card       | No plan changes; assumes invoices exist     | R1        | in-progress | specs/billing-payment-methods/ |
| R3 | Plan changes       | Upgrade/downgrade the subscription plan   | Uses R2's default payment method            | R1, R2    | planned | — |
```

그런 다음 각 조각을 단독으로 명세합니다. 예를 들어 **R2** 하위 기능의 `spec.md`는
역참조로 시작합니다.

```markdown
# Feature Specification: Billing — payment methods

**Input**: Parent roadmap: `specs/billing-portal/roadmap.md` → entry **R2**.
Let customers add, remove, and set a default payment method in the billing portal.
```

여기서부터 독자는 **R2**를 로드맵으로 거슬러 추적하고, 그것이 **R1**(청구서 이력,
이미 `done`)에 의존한다는 것과 **R3**(요금제 변경)이 그것을 기다리고 있다는 것을
알 수 있습니다. R1, R2, R3 순으로 만들면 로드맵이 에픽 전체의 형태를 보존하는 동안
모든 실행이 작게 유지됩니다.

## 자동화(선택 사항)

파일을 손으로 유지 관리하는 대신 로드맵 기록과 일관성 검사를 자동화하고 싶다면,
커뮤니티가 유지 관리하는
[Spec Roadmap 확장](https://github.com/srobroek/speckit-roadmap)이 그 방향을
탐구합니다. 서드파티 확장이며 필수는 아닙니다. 위의 수동 관례만으로도 충분합니다.
