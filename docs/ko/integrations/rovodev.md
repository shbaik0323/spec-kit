# RovoDev

설정하기 전에 [RovoDev](https://www.atlassian.com/software/rovo-dev)를 설치하세요.
통합 키는 `rovodev`이며, 런타임 디스패치는 `acli rovodev`를 사용합니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있지 않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration rovodev
```

기존 Spec Kit 프로젝트에서는 `specify integration install rovodev`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬과 프롬프트

이 통합은 `.rovodev/skills/`, `.rovodev/prompts/` 아래의 프롬프트 래퍼,
`.rovodev/prompts.yml`을 생성합니다.
스킬은 `speckit-<command>/SKILL.md` 레이아웃을 사용합니다. 에이전트의 채팅에서
`/speckit-specify`, 또는 설치된 bug 확장의 `/speckit-bug-assess`로 호출하세요.

프롬프트 래퍼에는 `use skill <name> $ARGUMENTS`가 들어 있습니다. 설정은 매니페스트
항목을 이름으로 병합합니다. 생성된 항목은 일치하는 항목을 대체하고, 일치하지 않는
사용자 항목은 순서를 유지합니다. 병합된 `prompts.yml`은 파일 전체로 추적됩니다.
설정 이후 변경되지 않았다면 이전에 병합된 사용자 항목이 들어 있어도 제거 시
삭제됩니다. 필요하다면 그 매니페스트를 백업하세요.

## 워크플로 디스패치

워크플로 스텝은 `acli rovodev run <prompt>`를 사용합니다. 실행 파일 오버라이드인
`SPECKIT_INTEGRATION_ROVODEV_EXECUTABLE`은 `rovodev` 하위 명령이 아니라 `acli`를
대체합니다. 구조화된 출력 요청은 결과 객체 스키마와 함께 `--output-schema`를
추가합니다. 워크플로의 `model` 설정은 전달되지 않습니다.
통합별 설정 옵션은 없습니다. 스킬 모드는 무조건 적용됩니다.
