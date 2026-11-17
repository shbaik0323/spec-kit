# ZCode

설정하기 전에 [ZCode](https://zcode.z.ai/)를 설치하세요.
통합 키는 `zcode`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration zcode
```

기존 Spec Kit 프로젝트에서는 `specify integration install zcode`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.zcode/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 달러 접두사로 호출하세요. 예: `$speckit-specify`, 또는 설치된
bug 확장의 `$speckit-bug-assess`.

선택적인 `--integration-options="--skills"` 플래그는 기본값을 다시 명시할
뿐입니다. 이 통합은 항상 스킬을 스캐폴딩합니다.

## 워크플로 디스패치

워크플로 스텝은 `zcode -p <prompt>`를 사용하고 구조화된 출력을 위해
`--output-format json`을 요청합니다. ZCode에는 호출별 모델 플래그가 없으므로
어댑터는 워크플로의 `model` 설정을 무시합니다. 세션 안에서 `/model`로 모델을
선택하세요.
