# Muse Code

설정하기 전에 [Muse Code](https://dev.meta.ai/docs/muse-code)를 설치하세요.
통합 키는 `muse`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있지 않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration muse
```

기존 Spec Kit 프로젝트에서는 `specify integration install muse`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.agents/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.
이 디렉터리는 다른 통합과 공유되므로, 그들이 생성한 스킬이 겹침 없이 공존할 수
있다고 가정하지 마세요.

선택적인 `--integration-options="--skills"` 플래그는 기본값을 다시 명시할
뿐입니다. 이 통합은 항상 스킬을 스캐폴딩합니다.

## 워크플로 디스패치

워크플로 스텝은 `muse exec <prompt>`를 사용하며, 설정된 경우 `--model`을, 구조화된
출력이 요청된 경우 `--json`을 함께 씁니다.
