# 커뮤니티 워크플로

커뮤니티 워크플로는 Spec Kit 스텝을 독립적으로 유지 관리되는 반복 가능한
프로세스로 합성합니다.

> [!NOTE]
> 메인테이너는 워크플로 코드, 스크립트, 의존성이 아니라 카탈로그 항목의 형식과
> 완전성을 확인합니다. 등재는 감사, 보증, 지원 약속이 아닙니다. 설치하고 사용하기
> 전에 소스, 필요한 통합, 부수 효과, 리뷰 게이트를 검토하세요.

## 사용 가능한 워크플로

[커뮤니티 워크플로 카탈로그](https://github.com/github/spec-kit/blob/main/workflows/catalog.community.json)의 항목:

| 워크플로 | 버전 | 설명 |
| --- | --- | --- |
| [ADO story lifecycle](https://github.com/praveenlavu/spec-kit-ado/tree/v0.3.2/workflows/ado-lifecycle) (`ado-lifecycle`) | 0.3.2 | 모든 Azure Boards 또는 PR 변경에 미리 보기, 사람의 게이트, 적용 단계를 두고 Azure DevOps 스토리를 열린 풀 리퀘스트까지 진행합니다 |
| [Guided SDD Pipeline](https://github.com/domattioli/spec-kit-workflow-pipeline) (`pipeline`) | 1.1.0 | clarify 게이트와 구현 후 수렴 루프를 두고 specify, clarify, plan, tasks, analyze, implement, converge를 연결합니다 |
| [Full SDD Cycle - no gates](https://github.com/clintcparker/speckit-addons/tree/yolo-v0.1.0/workflows/yolo) (`yolo`) | 0.1.0 | 리뷰 게이트 없이 specify, plan, tasks, implement를 실행합니다 |

이들은 Spec Kit에 번들된 워크플로와 구별되는 커뮤니티 정의입니다. 활성 소스와 공개된
릴리스를 확인하세요.

```bash
specify workflow catalog list
specify workflow search
specify workflow info <id> --versions
```

검색은 모든 활성 워크플로 카탈로그를 사용합니다. 내장 커뮤니티 소스는 탐색
전용입니다. 등재되었다고 해서 그 소스를 통해 `specify workflow add <id>`로 설치할 수
있게 되지는 않습니다. 릴리스를 검증한 뒤 고정된 URL을 명시적으로 설치하거나,
신뢰하고 설치를 허용하도록 별도로 설정한 카탈로그를 사용하세요.

## 워크플로 기여하기

버전이 지정된 워크플로 정의나 완전한 패키지를 게시하고,
`workflows/catalog.community.json`을 편집하는 초점이 좁은 PR을 제출하세요. 릴리스
메타데이터, 문서, 필요한 통합과 부수 효과, 그리고 해당하는 게이트 및 재개 동작을
포함한 종단 간 테스트 근거를 담으세요.

워크플로 정의는 제출 이슈 양식을 사용하지 않습니다. Workflow Step Type Submission
양식은 워크플로가 아니라 실행 가능한 사용자 정의 스텝 패키지를 위한 것입니다. 카탈로그
항목 형식과 릴리스 업데이트 절차는 게시 가이드가 다룹니다.

## 함께 보기

- [워크플로 개요와 번들된 정의](../workflows/overview.md)
- [워크플로 관리 CLI](../workflows/cli.md)
- [워크플로 개발하기](../workflows/developing.md)
- [워크플로 게시 및 제출](../workflows/publishing.md)
- [워크플로 카탈로그](../workflows/catalogs.md)
- [커뮤니티 워크플로 스텝 타입](workflow-steps.md)
