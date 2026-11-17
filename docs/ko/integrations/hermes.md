# Hermes

설정하기 전에 [Hermes Agent](https://github.com/NousResearch/hermes-agent)를 설치하세요.
통합 키는 `hermes`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있지 않습니다. 스킬이 같은 사용자의 홈
디렉터리에서 프로젝트 간에 공유됩니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration hermes
```

기존 Spec Kit 프로젝트에서는 `specify integration install hermes`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

프로젝트 로컬 스킬 통합과 달리 Hermes는 스킬을
`~/.hermes/skills/speckit-<command>/SKILL.md`로 전역 설치합니다.
Hermes의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.

비어 있는 프로젝트 로컬 `.hermes/skills/` 마커가 통합 감지를 가능하게 합니다.
생성된 코어 스킬 파일 자체는 프로젝트로 복사되지 않습니다.
선택적인 `--integration-options="--skills"` 플래그는 기본값을 다시 명시할
뿐입니다.

<a id="global-ownership-and-removal"></a>

## 전역 소유권과 제거

설정은 다른 프로젝트의 전역 Spec Kit 스킬 파일을 교체할 수 있습니다.

**제거는 편집되었더라도, `--force`가 없더라도 모든 전역 `speckit-*` 스킬
디렉터리를 삭제합니다.** 이 전역 파일들은 프로젝트 로컬 매니페스트의 수정 파일
검사로 보호되지 않습니다. Hermes를 제거하거나 다른 통합으로 전환하기 전에 사용자
정의를 백업하세요. 비어 있는 프로젝트 로컬 마커도 제거되며, Spec Kit이 아닌 전역
스킬은 남습니다.

## 워크플로 디스패치

워크플로 스텝은 `hermes chat -Q`를 사용합니다. 슬래시로 시작하는 프롬프트는
`-s`로 스킬을 선택하고, 나머지 텍스트는 `-q`로 전달됩니다. 그 밖의 프롬프트는
`-q`를 직접 사용합니다.
어댑터는 설정된 모델을 `-m`으로 전달하고 구조화된 출력을 위해 `--json`을
요청합니다.
