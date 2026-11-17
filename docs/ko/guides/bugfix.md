# 버그 수정 빠른 시작

기존 동작이 깨져서 근거에 기반한 진단, 범위가 정해진 수리, 원래 보고에 비춘
검증이 필요할 때 이 프로세스를 사용하세요. SDD 기능 워크플로를 먼저 실행할
필요는 없습니다.

번들로 제공되는 선택형 **bug** 확장은 작업을 **assess → fix → test**로
분리합니다. 각 버그는 `.specify/bugs/<slug>/` 아래에 진단, 변경 기록, 테스트
결과를 담은 디렉터리를 갖습니다.

## 설정

먼저 [Spec Kit을 설치](../installation.md)하고 버그가 있는 저장소를 초기화하세요.
이미 코드가 있는 저장소라면
[기존 프로젝트에 Spec Kit 도입하기](../how-to/existing-projects.md)를 따르세요.

초기화된 프로젝트 루트의 터미널에서 확장을 설치합니다.

```bash
specify extension add bug
```

그런 다음 그 디렉터리에서 코딩 에이전트를 실행하세요. 아래 예시는 GitHub
Copilot의 기본 스킬 모드(`--integration copilot`)를 사용합니다. 다른 에이전트는
각자의
[명령 호출 구문](../integrations/cli.md#command-invocation)으로 같은 단계를
노출합니다.
각 `/speckit-bug-*` 스킬을 에이전트의 채팅에서 따로따로 호출하고, 계속하기 전에
출력을 검토하세요. 이들은 터미널 명령이 아니라 에이전트 스킬입니다. 위의 터미널
명령은 확장을 설치할 뿐입니다.

## 1. 버그 평가

증상, 재현 단계, 기대 동작을 제공하세요. GitHub 이슈 URL이나 스택 트레이스도
쓸 수 있습니다. 버그에 짧고 재사용 가능한 이름을 정하세요.

```text
/speckit-bug-assess "Submitting the login form with an empty password crashes the app instead of showing a validation error." slug=login-crash
```

에이전트는 코드를 조사하고 `.specify/bugs/login-crash/assessment.md`를 작성합니다.
소스 코드는 변경하지 않습니다. 수정을 요청하기 전에 진단과 제안된 교정 방안을
검토하세요. 근거 없는 진단이나 버그가 아닌 보고로는 진행하지 마세요.

## 2. 평가된 원인 수정

같은 slug를 사용합니다.

```text
/speckit-bug-fix slug=login-crash
```

에이전트는 평가된 교정 방안을 적용하고 변경 사항을
`.specify/bugs/login-crash/fix.md`에 기록합니다. 소스 코드를 편집하는 단계는
이것뿐입니다. 새로운 근거로 인해 평가된 범위 밖의 작업이 필요해지면, 에이전트는
수리 범위를 말없이 넓히지 말고 그 편차를 기록해야 합니다.

## 3. 수정 테스트

```text
/speckit-bug-test slug=login-crash
```

에이전트는 재현 절차와 관련 테스트를 다시 실행한 다음
`.specify/bugs/login-crash/test.md`를 작성합니다. 이 단계는 근거를 기록합니다.
실패하는 테스트를 통과시키려고 소스 코드를 편집하지 않습니다.

| 판정 | 의미 | 할 일 |
| --- | --- | --- |
| `verified` | 검증 요구 사항이 성공적으로 수행됨 | 병합 전에 패치와 근거를 검토 |
| `partial` | 일부 검증을 완료하지 못함 | 누락된 환경이나 재현 근거를 제공하고 다시 테스트 |
| `failed` | 검증에서 남은 문제가 발견됨 | 그 근거로 진단이나 수정을 다시 살펴본 뒤 다시 테스트 |

원래의 재현 절차가 한 번도 수행되지 않았다면 테스트 스위트가 통과하는 것만으로는
충분하지 않습니다. 검토자가 증상부터 근거까지 수리 과정을 추적할 수 있도록 평가,
수정 기록, 테스트 보고서를 함께 보관하세요.

## 더 알아보기

- [버그 명령 레퍼런스](../reference/agentic-bugfix.md): 인자, slug 처리, 출력 계약.
- [SDD 빠른 시작](../quickstart.md): 작업이 수리가 아니라 새 기능일 때 사용합니다.
- [아이디어 평가 빠른 시작](assessment.md): 제안된 변경이 추진할 가치가 있는지 조사합니다.
