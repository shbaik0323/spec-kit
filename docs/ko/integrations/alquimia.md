# Alquimia AI

설정하기 전에 [Alquimia AI CLI](https://docs.alquimia.ai)를 설치하세요.
통합 키는 `alquimia`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration alquimia
```

기존 Spec Kit 프로젝트에서는 `specify integration install alquimia`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.alquimia/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.

생성된 스킬은 사용자와 모델 모두 호출할 수 있으며, 지원되는 코어 스킬에는 인라인
인자 힌트도 붙습니다.

## 워크플로 디스패치

워크플로 명령 및 프롬프트 스텝은 `alquimia -p <prompt>`를 사용하며, 설정된 경우
`--model`을, 구조화된 출력이 요청된 경우 `--output-format json`을 함께 씁니다.
통합별 설정 옵션은 없습니다.
