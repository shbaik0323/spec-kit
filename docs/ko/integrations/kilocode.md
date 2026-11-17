# Kilo Code

설정하기 전에 IDE에 [Kilo Code](https://github.com/Kilo-Org/kilocode)를 설치하세요.
통합 키는 `kilocode`이며, 별도의 CLI는 필요하지 않습니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration kilocode
```

기존 Spec Kit 프로젝트에서는 `specify integration install kilocode`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 명령

Markdown 명령은 `.kilo/commands/` 아래에 설치됩니다.
에이전트의 채팅에서 점 표기로 호출하세요. 예: `/speckit.specify`, 또는 설치된 bug
확장의 `/speckit.bug.assess`.

레거시 `.kilocode/workflows/` 설치는 등록 폴백으로 계속 지원됩니다.

업그레이드는 매니페스트로 추적되는 코어 명령을 `.kilo/commands/`로
마이그레이션합니다. 프리셋 명령 오버라이드가 레거시 디렉터리에 영향을 준다면
파일을 변경하기 전에 업그레이드가 거부됩니다. 프리셋을 제거하고, 업그레이드한
다음, 다시 설치하세요. 읽을 수 없는 프리셋 레지스트리도 그 마이그레이션을
차단합니다.

## 워크플로 제한

이 통합은 워크플로 명령 또는 프롬프트 스텝에 대한 CLI 디스패치를 지원하지
않습니다. IDE 명령만 설치하며 통합별 설정 옵션은 없습니다.
