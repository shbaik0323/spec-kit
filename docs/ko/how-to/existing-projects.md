# 기존 프로젝트에 Spec Kit 도입하기

Spec Kit을 사용하기 전에 기존 시스템을 명세로부터 다시 만들 필요는 없습니다.
저장소를 제자리에서 초기화하고, 중요한 규칙을 담고, 다음에 할 범위가 한정된
변경에 워크플로를 사용하세요.

## 1. 리뷰 가능한 기준점에서 시작하기

초기화하기 전에 기존 작업을 커밋하거나 stash하고 도입용 브랜치를 만드세요.
그러면 생성되는 모든 파일이 일반적인 코드 리뷰에서 보이게 됩니다.

사용하는 코딩 에이전트의 [통합 키](../integrations/index.md)를 고르세요.
그런 다음 저장소 루트에서 명령을 실행합니다.

```bash
specify init --here --force --integration <key>
```

`--here`는 현재 디렉터리를 대상으로 합니다. `--force`는 비어 있지 않은
디렉터리에서 초기화를 허용하며 충돌하는 관리 경로의 파일을 교체할 수 있으므로,
리뷰 가능한 기준점을 만든 뒤에만 사용하세요. 애플리케이션의 나머지를 삭제하지는
않습니다.

계속하기 전에 결과 diff를 검토하세요. 초기화는 공유 `.specify/` 프로젝트 파일과
선택한 통합에 필요한 명령 또는 스킬 파일을 추가합니다. 애플리케이션을 다시 쓰거나
기존 동작에 대한 명세를 추론하지 않습니다.

> [!NOTE]
> Git 초기화와 기능 브랜치는 선택 사항이며 **git** 확장이 관리합니다. 그
> 워크플로를 원한다면 `specify extension add git`으로 추가하세요.

## 2. 프로젝트 가드레일 담기

나머지 단계는 범위가 한정된 기능에 **SDD**를 도입하는 과정을 보여 줍니다. 첫
작업이 수리라면 초기화 후에 대신
[버그 수정 빠른 시작](../guides/bugfix.md)을 따를 수 있고, 제안을 평가하려면
[아이디어 평가 빠른 시작](../guides/assessment.md)을 사용하세요. 두 프로세스 모두
아래 SDD 단계를 먼저 완료할 필요가 없습니다.

아래 예시는 GitHub Copilot의 기본 스킬 모드(`/speckit-*`)를 사용합니다.
각 스킬을 터미널이 아니라 에이전트의 채팅에서 따로따로 호출하고, 다음 단계로
넘어가기 전에 결과를 검토하세요.

저장소에 이미 해당하거나 팀이 채택하기로 명시적으로 합의한 원칙으로
`/speckit-constitution`을 실행하세요.

```text
/speckit-constitution Preserve public API compatibility. Follow the existing
service boundaries. Every database migration must include a rollback plan.
Run the repository's established unit and integration test suites.
```

저장소의 README, 아키텍처 결정, 기여 가이드, CI 설정을 근거로 사용하세요.
컨스티튜션 템플릿을 채우기 위해 표준을 지어내지 마세요. 컨스티튜션은 이후의
계획과 분석을 규율하므로, 비현실적인 규칙은 유용한 제약 대신 소음을 만듭니다.

## 3. 범위가 한정된 첫 변경 고르기

독립적으로 리뷰할 수 있는 기능이나 현대화 조각으로 시작하세요. "기존 시스템 전체를
문서화"하는 것은 그 목록 자체가 의도한 산출물이 아니라면 첫 기능으로 삼지 마세요.

요청하는 결과와 그대로 유지되어야 하는 호환성 경계를 모두 설명하세요.

```text
/speckit-specify Add CSV export to the existing orders page. Preserve current
filters and authorization behavior. Export only the rows visible to the signed-in
user, and do not change the existing JSON API response.
```

코드베이스는 구현 컨텍스트로 남습니다. 새 `spec.md`는 여러분이 의도하는 변경을
정의하며, 모든 기존 동작에 대한 소급 명세가 아닙니다.

## 4. 저장소에 비추어 계획하기

일반적인 워크플로를 이어 가세요.

1. `/speckit-clarify`를 실행해 불확실한 동작과 호환성 요구 사항을 해소합니다.
2. `/speckit-plan`을 실행하고, 제안된 설계가 기존 아키텍처, 의존성, 테스트 관례를
   재사용하는지 확인합니다.
3. `/speckit-tasks`를 실행한 다음 `/speckit-analyze`로 구현 전에 일관성을
   확인합니다.
4. `/speckit-implement`를 실행하고 코드와 아티팩트 변경을 함께 검토합니다.
5. `/speckit-converge`를 실행해 남은 누락을 찾습니다. 태스크가 추가되면 기능이
   완성될 때까지 구현과 수렴을 반복합니다.

명령의 세부 사항과 선택적 품질 게이트는
[빠른 시작 가이드](../quickstart.md)와
[Agentic SDD 레퍼런스](../reference/agentic-sdd.md)를 참고하세요.

## 5. 스펙이 어떻게 나이 들지 결정하기

첫 변경 이후에는 팀이 어떤 기능 아티팩트를 남길지, 남긴 아티팩트를 어떻게 유지
관리할지 합의하세요. [명세 보존하기](persisting-specs.md) 페이지가 컨스티튜션 전용
모델에서 완료된 기능 아티팩트를 퇴역시키는 것을 포함해 그런 선택지를 비교합니다.
후속 변경을 전달할 때는 [스펙 발전시키기 가이드](evolving-specs.md)를 사용하세요.

## 기존 프로젝트 예시

[커뮤니티 워크스루](../community/walkthroughs.md)에는 .NET, Java, Go/React
코드베이스에 걸친 브라운필드 예시가 있습니다. 아키텍처 탐색과 브라운필드
부트스트랩을 위한 커뮤니티 확장은
[확장 카탈로그](../community/extensions.md)에 나열되어 있습니다.

## 함께 보기

- [명령 호출](../integrations/cli.md#command-invocation) — 다른 코딩 에이전트와
  통합 모드의 구문.
