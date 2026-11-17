# Antigravity

설정하기 전에 [Antigravity](https://antigravity.google/)를 설치하세요.
통합 키는 `agy`입니다. 스킬 레이아웃에는 Antigravity CLI v1.0.0 이상 또는
Antigravity IDE v2.0.0 이상이 필요합니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있지 않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration agy
```

기존 Spec Kit 프로젝트에서는 `specify integration install agy`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.agents/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.
이 디렉터리는 다른 통합과 공유되므로, 그들이 생성한 스킬이 겹침 없이 공존할 수
있다고 가정하지 마세요.

## 헤드리스 권한

자동 도구 승인은 기본적으로 비활성화되어 있습니다. 헤드리스 비대화형 실행에서는
대화형 승인 없이 도구 동작을 허가할 의도가 있을 때만
`SPECKIT_AGY_ALLOW_ALL_TOOLS=1` 또는
`SPECKIT_INTEGRATION_AGY_ALLOW_ALL_TOOLS=1`을 설정해
`--dangerously-skip-permissions`를 명시적으로 활성화하세요.

## 워크플로 디스패치

워크플로 스텝은 `agy --print <prompt>`를 사용합니다. Spec Kit은 스킬을 계속 찾을
수 있도록 프로젝트 루트와 함께 `--add-dir`을 제공하고, 설정된 `--model`을
전달하고, 구조화된 출력을 위해 `--output-format json`을 요청합니다.
통합별 설정 옵션은 없습니다.
