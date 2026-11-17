# Zed

설정하기 전에 [Zed](https://zed.dev/)를 설치하세요.
통합 키는 `zed`이며, 별도의 CLI는 필요하지 않습니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있지 않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration zed
```

기존 Spec Kit 프로젝트에서는 `specify integration install zed`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.agents/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.
이 디렉터리는 다른 통합과 공유되므로, 그들이 생성한 스킬이 겹침 없이 공존할 수
있다고 가정하지 마세요.

## 워크플로 제한

이 통합은 워크플로 명령 또는 프롬프트 스텝에 대한 CLI 디스패치를 지원하지
않습니다. 에디터 스킬만 설치하며 통합별 설정 옵션은 없습니다. `--skills` 플래그는
필요하지도, 선언되어 있지도 않습니다.
