# Junie

설정하기 전에 [Junie](https://junie.jetbrains.com/)를 설치하세요.
통합 키는 `junie`이며, Specify가 그 CLI를 확인합니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration junie
```

기존 Spec Kit 프로젝트에서는 `specify integration install junie`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 명령

Markdown 명령은 `speckit-specify.md` 같은 하이픈 파일명을 사용해
`.junie/commands/` 아래에 설치됩니다.
Junie의 채팅에서 점 표기 명령이 아니라 `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`로 호출하세요.

핸드오프 참조와 확장/프리셋 명령 내용도 하이픈 이름으로 정규화됩니다.

## 워크플로 디스패치

워크플로 스텝은 `junie --task <prompt>`를 사용하며, 설정된 경우 `--model`을,
구조화된 출력이 요청된 경우 `--output-format json`을 함께 씁니다.
통합별 설정 옵션은 없습니다.
