# Factory Droid

설정하기 전에 [Factory Droid](https://docs.factory.ai/cli/getting-started/overview)를 설치하세요.
통합 키는 `droid`입니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration droid
```

기존 Spec Kit 프로젝트에서는 `specify integration install droid`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬

스킬은 `.factory/skills/speckit-<command>/SKILL.md`로 설치됩니다.
에이전트의 채팅에서 호출하세요. 예: `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`.

생성된 스킬은 기본적으로 사용자와 모델 모두 호출할 수 있습니다.

## 사용자 호출 전용 스킬

Droid가 코어 Spec Kit 명령 스킬을 자동으로 호출하지 않게 하려면 다음과 같이 합니다.

```bash
specify init my-project --integration droid --integration-options="--no-model-invocation"
```

이 옵션은 코어 명령 스킬에 `disable-model-invocation: true`를 기록합니다.
Droid 1.0.0에서는 모델이 사용 가능한 스킬 목록에 이들을 더 이상 나열하지 않으므로
요청 없이 실행하지 않으며, `/speckit-<command>`와 워크플로 디스패치는 여전히 이들을
실행합니다. 이는 완전한 차단이 아닙니다. 스킬 이름을 언급하는 프롬프트(예: "use
the speckit-specify skill")는 여전히 모델이 `Skill` 도구를 통해 스킬을 호출하게
합니다. Factory의
[스킬 문서](https://docs.factory.com/cli/configuration/skills)는 이 동작을
설명하지 않습니다.

- 확장 스킬은 모델 호출 가능 상태로 남습니다. 필수 확장 훅이 모델의 호출에
  의존하기 때문입니다. 확장의 필수 훅이 코어 명령을 실행한다면(`command:`가 코어
  `speckit.<command>`인 경우) 이 옵션을 끄세요. 코어 스킬이 사용자 호출 전용인
  동안에는 모델이 그 훅을 실행할 수 없습니다.
- 이 옵션은 통합의 다른 옵션과 함께 저장되고
  `specify integration upgrade droid`에 의해 다시 적용됩니다. `upgrade`에
  `--integration-options`를 전달하면 저장된 모든 옵션이 대체됩니다.
  `--integration-options=""`는 옵션을 지웁니다. `specify init`을 다시 실행하는
  경우에도 마찬가지로 전달된 옵션만 저장됩니다.
- 프리셋이 오버라이드하거나 오버라이드를 중단한 코어 명령 스킬은, Droid를
  재설치하지 않고 프리셋 스킬을 다시 등록하는 명령에 의해 기본값으로 되돌려집니다.
  프리셋을 설치·제거·업데이트하는 명령(프리셋을 담은 `specify bundle` 명령 포함),
  `specify integration use droid`, 그리고 Droid가 설치되어 있지만 기본값이 아닐
  때의 `specify integration switch droid`가 해당합니다. 옵션을 다시 적용하려면
  Droid가 기본 통합인 상태에서
  `specify integration upgrade droid --force`를 실행하세요(그 명령들이 설치된 스킬
  파일을 수정하므로 `--force`가 필요하며, 이는 Spec Kit이 관리하는 파일에 대한
  로컬 편집을 덮어씁니다).

## 워크플로 디스패치

워크플로 스텝은 `droid exec <prompt>`를 사용하며, 적절한 경우 `--model`을 전달하고
`--output-format json`을 요청합니다. Spec Kit은 권한 우회 플래그를 추가하지
않습니다. 운영자는 `SPECKIT_INTEGRATION_DROID_EXTRA_ARGS`를 통해 권한 플래그를
명시적으로 설정할 수 있습니다. 그 인자들은 Spec Kit의 모델/출력 플래그 뒤에 오며,
중복 플래그 파싱에 따라 그것들을 재정의할 수 있습니다.
유일한 통합별 설정 옵션은 `--no-model-invocation`입니다.
