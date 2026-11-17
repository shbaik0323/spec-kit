# 워크플로

워크플로는 YAML 정의를 통해 여러 단계로 이루어진 Spec Kit 프로세스를
자동화합니다. 에이전트 명령, 프롬프트, 셸 작업, 제어 흐름, 사람의 리뷰 게이트를
반복 가능하고 재개 가능한 실행으로 엮습니다. `/speckit.*` 명령으로 수동으로 따르는
프로세스나 GitHub Actions 워크플로와는 별개입니다.

## 시작하기 전에

[Specify CLI](../installation.md)를 설치하고
[에이전트 통합](../integrations/index.md)으로 프로젝트를 초기화하세요. 에이전트
명령 및 프롬프트 스텝에는 CLI 실행이 가능한 통합이 필요합니다. 셸 전용 로컬 YAML
워크플로는 초기화된 프로젝트 밖에서도 실행할 수 있습니다.

설치하고 사용하기 전에 동반 스크립트, 호출되는 워크플로, 사용자 정의 스텝 패키지,
활성 오버레이를 포함해 워크플로의 소스를 검토하세요. 카탈로그 등재와 설치 권한은
코드 검토나 안전성의 증거가 아닙니다.

## 실행, 검토, 재개

```bash
specify workflow search
specify workflow add speckit
specify workflow run speckit --input spec="Build a kanban board"
```

번들된 SDD 워크플로는 specify, plan, tasks, implement를 실행하며, 명세와 계획
이후에 리뷰 게이트가 있습니다. 대화형 터미널에서는 게이트가 판정을 묻고, 그렇지
않으면 일시 정지합니다. 출력을 검토하고 같은 실행을 재개하세요.

```bash
specify workflow status <run_id>
specify workflow resume <run_id>
```

대화형 터미널 없이 일시 정지된 리뷰를 승인하려면, 출력을 검토한 뒤 해당하는 판정
입력을 제공하세요.

| 워크플로 | 리뷰 게이트 | 판정 입력 |
| --- | --- | --- |
| `speckit` | `review-spec` | `speckit_review_spec_verdict` |
| `speckit` | `review-plan` | `speckit_review_plan_verdict` |
| `bugfix` | `review-assessment` | `bug_review_assessment_verdict` |

```bash
specify workflow resume <sdd_run_id> --input speckit_review_spec_verdict=approve
specify workflow resume <sdd_run_id> --input speckit_review_plan_verdict=approve
specify workflow resume <bugfix_run_id> --input bug_review_assessment_verdict=approve
```

활성 리뷰에 해당하는 명령을 사용하세요. 이 문자열 입력은 선택 사항이며 기본값은
`""`입니다. 입력을 생략하거나 빈 값을 제공하면 대화형 터미널에서는 기존 프롬프트가,
그렇지 않으면 일시 정지가 유지됩니다. 명세를 승인한다고 해서 계획 리뷰가 승인되지는
않습니다.

이 입력들은 Specify CLI 0.15.1 이상에서 `speckit` 워크플로 버전 1.0.2와 `bugfix`
버전 1.0.1에서 사용할 수 있습니다. 새 입력 선언을 사용하려면 설치된 워크플로를
업데이트하고 새 실행을 시작하세요. 기존 실행은 시작할 때 저장된 워크플로 정의를
유지합니다. 설치된 워크플로를 업데이트해도 그 실행에 입력이 추가되지 않습니다.

각 실행은 자체 ID와 상태 디렉터리를 갖습니다. 완료된 실행 건은 구현을 다시 실행하지
않고 표현식 컨텍스트로 재생됩니다. 끝나지 않은 작업은 영속화된 상태에서
계속됩니다. 완료 체크포인트 이전에 중단된 부수 효과는 여전히 반복될 수 있습니다.

로컬 정의를 설치하지 않고 실행할 수도 있습니다.

```bash
specify workflow run ./workflow.yml --input spec="Build a kanban board"
```

## 번들된 워크플로

| ID | 용도 | 주요 입력 |
| --- | --- | --- |
| `speckit` | 명세 및 계획 리뷰 게이트를 갖춘 전체 SDD 전달 주기 | `spec` |
| `bugfix` | 버그 평가, 수정, 검증 | `report` |
| `assess` | 아이디어 접수, 조사, 정의, 구체화, 결정 | `idea` |

메타데이터, 입력, 스텝 그래프는 `specify workflow info <id>`를 사용하세요.
`speckit` 워크플로는 `spec`, 선택적인 `integration`, 위의 두 선택적 판정 입력을
선언하며 `scope` 입력은 선언하지 않습니다. 확장 명령을 사용하는 워크플로는 그
명령도 설치되어 있어야 합니다. 각 번들 정의의 요구 사항을 참고하세요.

## 워크플로 사용자 정의하기

새로운 자동화 프로세스를 위한 정의를 작성하거나, `workflow` 스텝을 통해 설치된 다른
워크플로를 호출하거나, 오버레이를 사용해 패키지를 편집하지 않고 설치된 워크플로를
사용자 정의하세요. 오버레이는 이름 있는 슬롯을 채우거나 스텝을 삽입, 교체, 제거할
수 있습니다. 기반 입력 스키마는 바꿀 수 없습니다.

## FAQ

### 같은 워크플로를 여러 번 실행할 수 있나요?

네. 실행은 독립적인 ID와 영속화된 상태를 갖습니다. `specify workflow status`는
실행을 나열하고, 실행 ID를 제공하면 그 상태를 보여 줍니다.

### 리뷰 게이트에서는 무슨 일이 일어나나요?

선언된 판정 입력이 프롬프트 없이 게이트를 결정할 수 있습니다. 그렇지 않으면
게이트는 TTY에서 프롬프트를 표시하거나 재개를 위해 일시 정지합니다. 거부는 설정에
따라 중단, 건너뛰기, 재시도를 위한 일시 정지가 될 수 있습니다.

<a id="who-maintains-workflows"></a>

### 워크플로는 누가 유지 관리하나요?

대부분의 워크플로는 작성자가 독립적으로 유지 관리합니다. 메인테이너는 워크플로
코드가 아니라 커뮤니티 카탈로그 항목의 형식과 완전성을 확인합니다. 그 코드를 검토,
감사, 보증, 지원하지 않습니다. 사용자는 설치하고 사용하기 전에 소스를 검증해야
합니다.

## 함께 보기

- [워크플로 CLI](cli.md)
- [워크플로 개발하기](developing.md)
- [워크플로 스텝 개요](../workflow-steps/overview.md)
- [내장 스텝 타입](../workflow-steps/types.md)
- [워크플로 오버레이](overlays.md)
- [카탈로그와 신뢰](catalogs.md)
- [워크플로 게시하기](publishing.md)
- [커뮤니티 워크플로](../community/workflows.md)
