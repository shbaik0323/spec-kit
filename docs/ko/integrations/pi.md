# Pi Coding Agent

설정하기 전에 [Pi Coding Agent](https://pi.dev)를 설치하세요.
통합 키는 `pi`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration pi
```

기존 Spec Kit 프로젝트에서는 `specify integration install pi`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 프롬프트

Markdown 프롬프트는 `.pi/prompts/` 아래에 설치됩니다.
에이전트의 채팅에서 점 표기로 호출하세요. 예: `/speckit.specify`, 또는 설치된 bug
확장의 `/speckit.bug.assess`.

## 워크플로 디스패치와 도구

워크플로 스텝은 `pi -p <prompt>`를 사용하며, 설정된 경우 `--model`을, 구조화된
출력이 요청된 경우 `--output-format`이 아니라 `--mode json`(JSONL 이벤트)을 함께
씁니다.
통합별 설정 옵션은 없습니다.

이 통합은 프롬프트 파일을 스캐폴딩할 뿐, MCP 서버를 설치하거나 설정하지 않습니다.
GitHub 도구가 필요한 명령은 Pi에서 그 도구를 별도로 설정해야 합니다.
[Pi 확장](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent#extensions)을
참고하세요.
