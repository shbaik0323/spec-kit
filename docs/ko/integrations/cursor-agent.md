# Cursor

설정하기 전에 [Cursor](https://cursor.sh/)를 설치하세요.
통합 키는 `cursor-agent`이며, IDE 설정에는 별도의 CLI가 필요하지 않습니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration cursor-agent
```

기존 Spec Kit 프로젝트에서는 `specify integration install cursor-agent`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.cursor/skills/speckit-<command>/SKILL.md`로 설치됩니다.
Cursor의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.

## 옵션과 워크플로 권한

선택적인 `--integration-options="--skills"` 플래그는 기본값을 다시 명시할
뿐입니다. 이 통합은 항상 스킬을 스캐폴딩합니다. 네이티브 이벤트는
`.cursor/hooks.json`을 사용하며, `--events false`는 Spec Kit 이벤트 핸들러를
비활성화합니다.
[런타임 이벤트](cli.md#runtime-events)를 참고하세요.

워크플로 명령 및 프롬프트 스텝에는 추가로 `cursor-agent` CLI가 필요합니다.
디스패치는 `-p --trust --approve-mcps --force`를 사용합니다. 이 플래그들은
워크스페이스 신뢰 프롬프트를 우회하고 MCP 로딩과 도구 동작을 승인합니다. 그런
권한을 부여할 의도가 있는 프로젝트에서만 워크플로를 실행하세요. 설정된 `--model`과
요청된 `--output-format json`은 전달됩니다.
