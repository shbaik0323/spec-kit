# Qoder CLI

설정하기 전에 [Qoder CLI](https://qoder.com/cli)를 설치하세요.
통합 키는 `qodercli`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration qodercli
```

기존 Spec Kit 프로젝트에서는 `specify integration install qodercli`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.qoder/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.

스킬 레이아웃은 레거시 `.qoder/commands/` 스캔을 대체합니다. 활성화된 확장이 다시
등록될 때, 이전의 평면 확장 명령 파일은 대체 스킬 파일이 존재한 뒤에만
퇴역합니다. 다시 등록하기 전에 그 레거시 확장 명령에 대한 편집을 백업하세요. 그
퇴역 과정은 콘텐츠 해시를 비교하지 않습니다.

## 워크플로 디스패치

워크플로 스텝은 `qodercli -p <prompt>`를 사용하며, 설정된 경우 `--model`을,
구조화된 출력이 요청된 경우 `--output-format json`을 함께 씁니다.
통합별 설정 옵션은 없습니다. 스킬 모드는 무조건 적용됩니다.
