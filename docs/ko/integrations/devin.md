# Devin for Terminal

설정하기 전에 [Devin for Terminal](https://cli.devin.ai/docs)을 설치하세요.
통합 키는 `devin`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있지 않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration devin
```

기존 Spec Kit 프로젝트에서는 `specify integration install devin`을 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.devin/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.

## 옵션과 워크플로 디스패치

선택적인 `--integration-options="--skills"` 플래그는 기본값을 다시 명시할
뿐입니다. 이 통합은 항상 스킬을 스캐폴딩합니다. 네이티브 이벤트는
`.devin/hooks.v1.json`을 사용하며, `--events false`는 Spec Kit 이벤트 핸들러를
비활성화합니다.
[런타임 이벤트](cli.md#runtime-events)를 참고하세요.

워크플로 스텝은 `devin -p <prompt>`를 사용하고 설정된 `--model`을 전달합니다.
Devin의 어댑터는 구조화된 출력이 요청되어도 일반 텍스트 stdout을 반환하며, JSON
출력 플래그를 추가하지 않습니다.
