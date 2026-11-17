# Kimi Code

설정하기 전에 [Kimi Code](https://code.kimi.com/)를 설치하세요.
통합 키는 `kimi`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있지 않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration kimi
```

기존 Spec Kit 프로젝트에서는 `specify integration install kimi`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.kimi-code/skills/speckit-<command>/SKILL.md`로 설치됩니다.
Kimi의 채팅에서 skill-콜론 접두사로 호출하세요. 예: `/skill:speckit-specify`, 또는
설치된 bug 확장의 `/skill:speckit-bug-assess`.

선택적인 `--integration-options="--skills"` 플래그는 기본값을 다시 명시할
뿐입니다. 이 통합은 항상 스킬을 스캐폴딩합니다.

<a id="legacy-migration"></a>

## 레거시 마이그레이션

오래된 설치를 업그레이드할 때 `--integration-options="--migrate-legacy"`를
전달하면 스킬을 `.kimi/skills/`에서 `.kimi-code/skills/`로 옮깁니다.
마이그레이션은 `speckit.xxx` 같은 점 표기 스킬 이름을 `speckit-xxx` 같은 하이픈
이름으로도 변환합니다.

```bash
specify integration upgrade kimi --integration-options="--migrate-legacy"
```

설정은 새 스킬을 먼저 씁니다. 레거시 디렉터리는 새 대상이 존재하지 않을 때만
옮겨집니다. 대상이 이미 존재하면, 두 `SKILL.md` 파일이 바이트 단위로 동일하고
레거시 디렉터리에 추가 파일이 없을 때만 이전 디렉터리가 제거됩니다. 내용이 다른
레거시 콘텐츠와 추가 사용자 파일은 수동 조정을 위해 유지됩니다. 심볼릭 링크된
레거시 경로는 건너뛰고, 심볼릭 링크된 대상 구성 요소는 거부됩니다.

<a id="legacy-cleanup-on-uninstall"></a>

## 제거 시 레거시 정리

일반적인 매니페스트 추적 파일은 공통 수정 파일 보호를 사용합니다. 그에 더해 해체
과정은 생성된 `metadata.author`와 `metadata.source`로 식별되는 레거시
`.kimi/skills/speckit-*` 및 `speckit.*` 디렉터리를 제거합니다.
이 레거시 정리는 기록된 해시를 비교하거나 `--force`를 요구하지 않습니다. 제거하기
전에 편집한 레거시 스킬과 그 디렉터리 안의 추가 파일을 백업하세요.

## 워크플로 디스패치

워크플로 스텝은 `/skill:speckit-<command>` 호출과 함께 `kimi -p <prompt>`를
사용하고 설정된 `--model`을 전달합니다. 구조화된 출력 요청은 지원되지 않는 `json`
값이 아니라 `--output-format stream-json`(JSONL 이벤트)을 사용합니다.
