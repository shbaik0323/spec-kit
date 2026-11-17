# Tabnine CLI

설정하기 전에 [Tabnine CLI](https://docs.tabnine.com/main/getting-started/tabnine-cli)를 설치하세요.
통합 키는 `tabnine`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration tabnine
```

기존 Spec Kit 프로젝트에서는 `specify integration install tabnine`을 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 명령

TOML 명령은 `.tabnine/agent/commands/speckit.<command>.toml`로 설치됩니다.
에이전트의 채팅에서 점 표기로 호출하세요. 예: `/speckit.specify`, 또는 설치된 bug
확장의 `/speckit.bug.assess`.

## 런타임 이벤트와 워크플로 디스패치

네이티브 이벤트는 `.tabnine/agent/settings.json`을 사용하며, 프롬프트 제출 및 중지
이벤트를 `BeforeAgent`와 `AfterAgent`에 매핑합니다. 훅 타임아웃은 밀리초로
변환됩니다. `--integration-options="--events false"`는 Spec Kit 이벤트 핸들러를
비활성화합니다. [런타임 이벤트](cli.md#runtime-events)를 참고하세요.

워크플로 스텝은 `tabnine -p <prompt>`를 사용하며, 모델이 설정된 경우 `-m`을,
구조화된 출력이 요청된 경우 `--output-format json`을 함께 씁니다.
