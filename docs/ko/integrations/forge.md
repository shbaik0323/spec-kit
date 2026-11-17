# Forge

설정하기 전에 [Forge](https://forgecode.dev/)를 설치하세요.
통합 키는 `forge`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있지 않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration forge
```

기존 Spec Kit 프로젝트에서는 `specify integration install forge`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 명령

Markdown 명령은 `speckit.specify.md` 같은 점 표기 파일명을 사용해
`.forge/commands/` 아래에 설치됩니다. 프런트매터의 명령 이름은 `speckit-specify`
처럼 하이픈으로 연결됩니다. 파일명과 호출 이름이 다릅니다.
에이전트의 채팅에서 점 표기 명령이 아니라 `/speckit-specify`, 또는 설치된 bug
확장의 `/speckit-bug-assess`로 호출하세요.

Forge 템플릿은 인자에 `{{parameters}}`를 사용합니다. Spec Kit은 코어 설정 중에
호환되지 않는 `handoffs` 프런트매터를 제거합니다.

## 워크플로 디스패치

워크플로 스텝은 `forge -p <prompt>`를 사용합니다. 어댑터는 설정된 `model`을
전달하거나 구조화된 출력 플래그를 추가하지 않습니다. 모델은 Forge 자체 설정에서,
예를 들어 `forge config set model`로 선택하세요.
통합별 설정 옵션은 없습니다.
