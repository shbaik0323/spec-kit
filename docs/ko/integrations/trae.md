# Trae

설정하기 전에 IDE에 [Trae](https://www.trae.ai/)를 설치하세요.
통합 키는 `trae`이며, 별도의 CLI는 필요하지 않습니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration trae
```

기존 Spec Kit 프로젝트에서는 `specify integration install trae`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.trae/skills/speckit-<command>/SKILL.md`로 자동 설치됩니다.
에이전트의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.

선택적인 `--integration-options="--skills"` 플래그는 기본값을 다시 명시할
뿐입니다. 이 통합은 레거시 명령 파일이 아니라 항상 스킬을 스캐폴딩합니다.

## 워크플로 제한

이 통합은 워크플로 명령 또는 프롬프트 스텝에 대한 CLI 디스패치를 지원하지
않습니다. 지원되는 경로는 IDE 스킬 인터페이스입니다.
