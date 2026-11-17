# Amp

설정하기 전에 [Amp CLI](https://ampcode.com/)를 설치하세요.
통합 키는 `amp`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있지 않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration amp
```

기존 Spec Kit 프로젝트에서는 `specify integration install amp`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 명령

Markdown 명령은 `.agents/commands/` 아래에 설치됩니다.
에이전트의 채팅에서 점 표기로 호출하세요. 예: `/speckit.specify`, 또는 설치된 bug
확장의 `/speckit.bug.assess`.

## 워크플로 디스패치

워크플로 스텝은 `amp --execute <prompt>`를 사용하며, 구조화된 출력이 요청되면
`--stream-json`을 추가합니다. 이 통합은 워크플로의 `model` 설정을 전달하지
않습니다. Amp의 `--mode`는 모델이 아니라 에이전트 모드를 선택합니다.
통합별 설정 옵션은 없습니다.
