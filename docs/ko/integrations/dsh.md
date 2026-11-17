# DeepSeek Harness

설정하기 전에 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)를 설치하세요.
통합 키는 `dsh`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration dsh
```

기존 Spec Kit 프로젝트에서는 `specify integration install dsh`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.dsh/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.

## 워크플로 디스패치

워크플로 스텝은 `dsh --profile headless <prompt>`를 사용합니다. 어댑터는 워크플로의
`model`과 구조화된 출력 요청을 무시합니다. 헤드리스 프로필은 마지막 어시스턴트
메시지를 일반 텍스트로 반환합니다.
통합별 설정 옵션은 없습니다.
