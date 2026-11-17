# Kiro CLI

설정하기 전에 [Kiro CLI](https://kiro.dev/docs/cli/)를 설치하세요.
통합 키는 `kiro-cli`입니다. 실행 파일 호환성 검사는 `kiro`도 받아들이지만,
`kiro`는 등록된 통합 키가 아닙니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration kiro-cli
```

기존 Spec Kit 프로젝트에서는 `specify integration install kiro-cli`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 프롬프트

Markdown 프롬프트는 `.kiro/prompts/` 아래에 설치됩니다.
Spec Kit은 `speckit.specify` 같은 점 표기 이름을 사용합니다. 선택하려면 Kiro의
[프롬프트 인터페이스](https://kiro.dev/docs/cli/chat/manage-prompts/)를 사용하세요.

Kiro CLI는 파일 기반 프롬프트에서 `$ARGUMENTS`를 치환하지 않습니다. 따라서 Spec
Kit은 모델에게 대화에서 인자를 가져오라고 지시하는 산문 폴백을 렌더링합니다.
[github/spec-kit#1926](https://github.com/github/spec-kit/issues/1926)을
참고하세요.

## 워크플로 디스패치

워크플로 스텝은 `-p <prompt>`와 함께 `kiro-cli` 실행 파일을 사용하며, 적절한 경우
`--model`을 전달하고 `--output-format json`을 요청합니다.
도구 감지는 `kiro` 실행 파일도 인식하지만, 어댑터의 기본 디스패치 실행 파일은
`kiro-cli`로 남습니다. `kiro`만 사용할 수 있다면 워크플로 디스패치를 위해
`SPECKIT_INTEGRATION_KIRO_CLI_EXECUTABLE`을 그 경로로 설정하세요.
통합별 설정 옵션은 없습니다.
