# Goose

설정하기 전에 [Goose](https://goose-docs.ai/)를 설치하세요.
통합 키는 `goose`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있지 않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration goose
```

기존 Spec Kit 프로젝트에서는 `specify integration install goose`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 레시피

Goose는 `.goose/recipes/speckit.<command>.yaml`로 설치되는 YAML 레시피를
사용합니다. Spec Kit 워크플로 디스패치는 `/speckit.<command>` 호출을 레시피 실행으로
변환하며, 뒤따르는 인자를 레시피의 `args` 매개변수로 전달합니다.
레시피가 스킬이라고 가정하지 말고 대화형 실행에는 Goose의 레시피 인터페이스를
사용하세요.

## 워크플로 디스패치

디스패치는 `/speckit.<command>` 프롬프트에는 `goose run --recipe <path>`를, Spec
Kit이 아닌 슬래시 명령을 포함한 그 밖의 프롬프트 텍스트에는
`goose run -t <prompt>`를 사용합니다. 설정된 `--model`과 요청된
`--output-format json`은 전달됩니다.

`SPECKIT_INTEGRATION_GOOSE_EXTRA_ARGS`에서 `--model`, `--recipe`,
`--output-format` 같은 단일 값 플래그를 반복하지 마세요. Goose는 마지막 값을
사용하는 대신 중복된 플래그를 거부합니다.
통합별 설정 옵션은 없습니다.
