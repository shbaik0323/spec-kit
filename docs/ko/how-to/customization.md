# Spec Kit을 여러분의 일하는 방식에 맞추기

세 가지 퍼스트파티 프로세스는 출발점이지 한계가 아닙니다. 내장 SDD 프로세스를
유지하거나, 버그 수정이나 평가를 추가하거나, 프로세스를 조직에 맞추거나, 완전히
다른 프로세스를 가져오세요.

## 알맞은 구성 요소 고르기

| 목표 | 사용할 것 |
| --- | --- |
| Spec Kit을 코딩 에이전트에 연결하거나 명령/스킬 전달 방식 조정 | 통합 |
| 새 명령, 기능, 프로세스 추가 | 확장 |
| 외부 도구나 서비스 연동 | 확장 |
| 스펙, 계획, 태스크의 형식이나 용어 변경 | 프리셋 |
| 기존 템플릿에 조직 또는 규제 표준 적용 | 프리셋 |
| 재사용 가능한 도메인 특화 템플릿 배포 | 오버라이드에는 프리셋, 새 명령에 딸린 템플릿에는 확장 |
| 단일 프로젝트에서 일회성 템플릿 조정 | 프로젝트 로컬 오버라이드 |
| 여러 단계로 이루어진 프로세스 자동화 | 워크플로 |
| 워크플로용 사용자 정의 실행 스텝 타입 추가 | 워크플로 스텝 패키지 |
| 한 번의 작업으로 완전한 역할 기반 구성 프로비저닝 | 번들 |

통합은 명령과 스킬이 코딩 에이전트에 어떻게 전달되는지를 결정하며, 외부 도구나
서비스에 연결하는 확장과는 구별됩니다. 지원되는 에이전트와 설정은
[통합 가이드](../integrations/index.md)를, 어댑터를 개발하려면
[통합 기여하기](../integrations/contributing.md)를 참고하세요.

워크플로 스텝 패키지는 워크플로에서 사용할 수 있는 스텝 타입을 확장합니다. 작성과
실행은 [사용자 정의 스텝 패키지](../workflow-steps/developing.md#custom-step-packages)를,
설치는 [스텝 CLI](../workflow-steps/cli.md#step-management)를, 탐색과 기여 안내는
[커뮤니티 워크플로 스텝](../community/workflow-steps.md)을 참고하세요. 실행 가능한
패키지는 설치하거나 사용하기 전에 검토하세요.

## 확장: 기능 추가

확장은 명령, 템플릿, 스크립트, 훅을 통해 **Spec Kit이 할 수 있는 일**을
넓힙니다. 도메인 특화 프로세스, 외부 연동, 코어를 넘어서는 개발 단계를 추가할 수
있습니다.

```bash
specify extension search
specify extension add <extension-name>
```

예로는 Jira 연동, 구현 후 코드 리뷰, V-모델 테스트 추적성, 프로젝트 상태 진단이
있습니다. 번들로 제공되는 [bug](../guides/bugfix.md)와
[assess](../guides/assessment.md) 확장이 퍼스트파티 예시입니다.
[커뮤니티 확장](../community/extensions.md)을 둘러보거나, 관리에는
[확장 CLI 레퍼런스](../extensions/cli.md)를, 설정에는
[확장 사용자 가이드](../extensions/overview.md#configuration)를 사용하세요.

## 프리셋: 프로세스의 동작 방식 변경

프리셋은 코어**와 설치된 확장**이 제공하는 템플릿과 명령을 오버라이드합니다. 새
도구 없이도 만들어지는 아티팩트와 지침을 사용자 정의합니다.

```bash
specify preset search
specify preset add <preset-name>
```

프리셋을 사용해 규제 추적성을 요구하거나, 테스트 우선 태스크 순서를 강제하거나,
계획에 보안 검토 게이트를 추가하거나, 워크플로를 현지화하세요. 프리셋은 방법론을
애자일, 칸반, 워터폴, JTBD(jobs-to-be-done), 도메인 주도 설계에 맞출 수도
있습니다.
[해적 말투 데모](https://github.com/mnriem/spec-kit-pirate-speak-preset-demo)는
용어를 얼마나 폭넓게 바꿀 수 있는지 보여 줍니다.

여러 프리셋을 우선순위와 함께 쌓을 수 있습니다. 예시는
[커뮤니티 프리셋](../community/presets.md)을, 쌓기와 합성은
[프리셋 가이드](../presets/overview.md)를, 설치와 관리는
[CLI 레퍼런스](../presets/cli.md)를 참고하세요.

<a id="project-local-overrides-and-resolution"></a>

## 프로젝트 로컬 오버라이드와 해석

일회성 프로젝트 사용자 정의라면 재사용 가능한 프리셋을 만들지 말고 템플릿
오버라이드를 `.specify/templates/overrides/`에 두세요.
기본 대체 순서는 다음과 같습니다.

1. 프로젝트 로컬 오버라이드.
2. 설치된 프리셋(우선순위 순).
3. 설치된 확장(우선순위 순).
4. `.specify/templates/`의 Spec Kit 코어 템플릿.

템플릿은 필요할 때 해석되며 기본적으로 첫 번째로 일치하는 것을 사용합니다.
명령은 다릅니다. 확장이나 프리셋을 설치하면 명령 파일이 활성 통합의 디렉터리에
구체화됩니다. 에이전트는 명령을 호출할 때마다 그 스택을 다시 해석하지 않습니다.
오버라이드하는 컴포넌트를 제거하면 조정(reconciliation)을 통해 남아 있는 명령
레이어가 복원됩니다.

사용자 정의가 없으면 코어 기본값이 적용됩니다. 경로, 우선순위,
prepend/append/wrap 합성에 대한 권위 있는 안내는
[파일 해석 레퍼런스](../presets/api.md#file-resolution)입니다.

## 번들: 역할 기반 구성

번들은 선별된 확장, 프리셋, 워크플로, 워크플로 스텝의 집합을 역할이나 팀을 위한
하나의 버전 지정 구성으로 패키징합니다. `bundle.yml` 매니페스트가 컴포넌트를
고정하며 통합을 대상으로 지정할 수 있습니다. 통합에 중립적인 번들은 프로젝트의
활성 통합을 상속합니다.

```bash
specify bundle search
specify bundle info <bundle-id>
specify bundle install <bundle-id>
```

설치하기 전에 `info`를 확인해 해석된 컴포넌트 집합을 살펴보세요. 사용 가능한
퍼스트파티 구성은 [번들 개요](../bundles/overview.md)를, 업데이트·제거·오프라인
한계는 [CLI 레퍼런스](../bundles/cli.md)를 참고하세요. 작성은
[번들 개발하기](../bundles/developing.md)와
[번들 게시하기](../bundles/publishing.md)부터 시작하세요.

## 사용자 정의 공유하기

커뮤니티 컴포넌트는 독립적으로 유지 관리됩니다. 설치 전에 소스 코드를 검토하고
여러분의 판단에 따라 사용하세요. 컴포넌트를 탐색하거나 직접 게시하려면
[커뮤니티 가이드](../community/overview.md)를, 프로세스를 자동화하려면
[워크플로 가이드](../workflows/overview.md)를 방문하세요.
