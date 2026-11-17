# Cline

설정하기 전에 IDE에 [Cline](https://github.com/cline/cline)을 설치하세요.
통합 키는 `cline`이며, 별도의 CLI는 필요하지 않습니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration cline
```

기존 Spec Kit 프로젝트에서는 `specify integration install cline`을 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 명령

Markdown 워크플로는 `speckit-specify.md` 같은 하이픈 파일명을 사용해
`.clinerules/workflows/` 아래에 설치됩니다.
Cline의 채팅에서 점 표기 명령이 아니라 `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`로 호출하세요.

핸드오프 참조와 확장/프리셋 명령 내용도 하이픈 명령 이름으로 정규화되므로, 확장
명령의 `/speckit.git.commit`은 `/speckit-git-commit`이 됩니다.
`.specify/extensions/git/commands/speckit.git.commit.md`,
`C:\speckit.foo.md`, `C:\Program Files (x86)\speckit.foo.md`,
`%TEMP%\speckit.foo.md`, `./commands[old]/speckit.foo.md`처럼 경로 안에 있는
`speckit.*.md` 파일 이름은 점과 `.md` 확장자를 유지합니다. `/speckit.foo.md`처럼
단독이거나 슬래시가 앞에 붙은 토큰은 명령 참조로 취급되어 하이픈으로 바뀝니다.
코어 명령 내용은 이런 식으로 다시 쓰이지 않습니다.

## 워크플로 제한

이 통합은 워크플로 명령 또는 프롬프트 스텝에 대한 CLI 디스패치를 지원하지
않습니다. IDE 워크플로만 설치하며 통합별 설정 옵션은 없습니다.
