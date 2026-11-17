# Grok Build

설정하기 전에 [Grok Build](https://docs.x.ai/build/overview)를 설치하세요.
통합 키는 `grok`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration grok
```

기존 Spec Kit 프로젝트에서는 `specify integration install grok`을 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.grok/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.

## 워크플로 권한

워크플로 스텝은 도구 실행을 자동으로 승인하는
`grok -p <prompt> --always-approve`를 사용합니다. 그 권한을 의도한 곳에서만
워크플로를 실행하세요. 설정된 `--model`과 요청된 `--output-format json`은
전달됩니다.
통합별 설정 옵션은 없습니다.
