# MiniMax Code

설정하기 전에 [MiniMax Code](https://github.com/MiniMax-AI/minimax-code)를 설치하세요.
통합 키는 `mcode`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration mcode
```

기존 Spec Kit 프로젝트에서는 `specify integration install mcode`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.minimax/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.

선택적인 `--integration-options="--skills"` 플래그는 기본값을 다시 명시할
뿐입니다. 이 통합은 항상 스킬을 스캐폴딩합니다.

## 헤드리스 권한

헤드리스 워크플로 디스패치는 도구 동작이 헤드리스 권한 게이트에 막히지 않도록
`--permission full`을 사용합니다. MiniMax Code의 더 엄격한 정책을 사용하려면
`SPECKIT_INTEGRATION_MCODE_EXTRA_ARGS="--permission smart"`를 설정하세요.

디스패치는 `mcode exec <prompt>`를 사용합니다. 설정된 `--model`과 요청된
`--output-format json`은 전달됩니다.
