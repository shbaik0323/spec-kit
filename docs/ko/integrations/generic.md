# Generic: 직접 가져온 에이전트 사용

내장 통합이 다루지 않는 AI 코딩 에이전트에는 `generic` 통합을 사용하세요.
`--commands-dir`로 프로젝트 로컬 출력 디렉터리를 제공합니다.
Spec Kit은 이 통합에 특정 에이전트 CLI를 요구하지 않습니다.

## 다중 설치 안전성

이 통합은 동적 출력 경로를 사용하므로 다중 설치 안전으로 선언되어 있지 않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration generic --integration-options="--commands-dir .myagent/cmds"
```

기존 Spec Kit 프로젝트의 경우:

```bash
specify integration install generic --integration-options="--commands-dir .myagent/cmds"
```

활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

<a id="output-options-and-invocation"></a>

## 출력 옵션과 호출

| 옵션 | 설명 |
| --- | --- |
| `--commands-dir <path>` | 필수. 명령 또는 스킬을 위한 프로젝트 로컬 출력 디렉터리. |
| `--skills` | 평면적인 `speckit.<command>.md` 파일 대신 `speckit-<command>/SKILL.md` 디렉터리를 씁니다. |

기본 명령 참조는 `/speckit.specify`와 `/speckit.bug.assess`를 사용합니다.
스킬 모드는 명령 참조와 다음 단계 안내를 `/speckit-specify`와
`/speckit-bug-assess`로 바꿉니다.

```bash
specify integration install generic --integration-options="--commands-dir .myagent/skills --skills"
```

에이전트가 선택한 파일 레이아웃과 호출 방식을 지원해야 합니다. Spec Kit은
에이전트에 네이티브 명령 탐색 기능을 추가하지 않습니다.

## 확장

`generic`이 활성 통합이 되면 `specify extension add`는 설정된 `--commands-dir`에
확장 명령을 `--skills` 여부에 따라 명령 파일 또는 스킬로 등록합니다.
`specify extension remove`는 코어 명령, 사용자 파일, 편집된 확장 파일은 그대로
두고 변경되지 않은 확장 소유 아티팩트를 제거합니다.

## 워크플로 제한

generic 통합은 워크플로 명령 또는 프롬프트 스텝에 대한 CLI 디스패치를 지원하지
않습니다. 출력 디렉터리를 선택한다고 해서 실행 파일이 설정되거나 Spec Kit이 임의의
에이전트 CLI를 호출하는 방법을 알게 되지는 않습니다.
