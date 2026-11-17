# opencode

설정하기 전에 [opencode](https://opencode.ai/)를 설치하세요.
통합 키는 `opencode`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration opencode
```

기존 Spec Kit 프로젝트에서는 `specify integration install opencode`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 명령

Markdown 명령은 `.opencode/commands/` 아래에 설치됩니다.
에이전트의 채팅에서 점 표기로 호출하세요. 예: `/speckit.specify`, 또는 설치된 bug
확장의 `/speckit.bug.assess`.

레거시 `.opencode/command/` 설치는 정식 디렉터리가 존재하지 않을 때 등록 폴백으로
남습니다.

## 런타임 이벤트와 워크플로 디스패치

네이티브 이벤트는 JSON 훅 맵이 아니라 `.opencode/plugin/speckit-events.ts`를
사용합니다. Spec Kit은 플러그인 참조를 `opencode.json`에 병합합니다.
`--integration-options="--events false"`는 Spec Kit 이벤트 핸들러를
비활성화합니다.
[런타임 이벤트](cli.md#runtime-events)를 참고하세요.

워크플로 스텝은 `opencode run`을 사용합니다. 슬래시로 시작하는 프롬프트는
`--command <name>`이 되고 나머지 텍스트가 메시지가 됩니다. 그 밖의 프롬프트는
메시지 텍스트로 전달됩니다. 어댑터는 설정된 모델을 `-m`으로 전달하고 구조화된
출력을 위해 `--format json`을 요청합니다.
