# SHAI (OVHcloud)

설정하기 전에 [SHAI](https://github.com/ovh/shai)를 설치하세요.
통합 키는 `shai`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration shai
```

기존 Spec Kit 프로젝트에서는 `specify integration install shai`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 명령

Markdown 명령은 `speckit.specify.md` 같은 점 표기 이름을 사용해
`.shai/commands/` 아래에 설치됩니다.

## 워크플로 제한

워크플로 `command` 및 `prompt` 스텝은 SHAI를 통해 실행할 수 없습니다. SHAI의 CLI는
프롬프트 텍스트를 인자로, stdin으로, 또는 `shai agent`를 통해 받지만 이를 자동 수정
에이전트에 넘기고 `.shai/commands/`를 로드하지 않으므로 그 스텝들은 실패합니다.

어댑터는 설치된 명령을 실행하지 않고 성공을 보고하는 대신 CLI 디스패치를
명시적으로 거부합니다. 통합별 설정 옵션은 없습니다.
