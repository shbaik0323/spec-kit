# IBM Bob

설정하기 전에 [IBM Bob](https://www.ibm.com/products/bob)을 설치하세요.
통합 키는 `bob`이며, 별도의 CLI는 필요하지 않습니다.

## 다중 설치 안전성

이 통합은 다중 설치 안전으로 선언되어 있지 않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration bob
```

기존 Spec Kit 프로젝트에서는 `specify integration install bob`을 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 기본 스킬 레이아웃

스킬은 `.bob/skills/speckit-<command>/SKILL.md`로 설치됩니다.
Bob의 스킬은 의도에 의해 활성화됩니다. Bob에게 `speckit-specify` 스킬을, 또는
설치된 bug 확장의 `speckit-bug-assess`를 사용하라고 요청하세요. Spec Kit은 스킬
모드 명령 디스패치를 위해 `/speckit-<command>` 참조를 렌더링하지만, 그렇다고 각
스킬이 별도로 등록된 IDE 슬래시 명령이라는 뜻은 아닙니다.

<a id="legacy-commands-and-migration"></a>

## 레거시 명령과 마이그레이션

더 이상 권장되지 않는 Bob 1.x 레이아웃(`.bob/commands/*.md`)을 대신 스캐폴딩하려면
`--integration-options="--legacy-commands"`를 전달하세요. 이 플래그는 향후
릴리스에서 제거될 예정입니다.

기존 레거시 설치는 다음으로 마이그레이션할 수 있습니다.

```bash
specify integration upgrade bob --integration-options="--skills"
```

이 명령은 설치를 스킬 레이아웃으로 변환하고 이전 명령 파일을 제거합니다. 프리셋
오버라이드가 설치되어 있으면, 프리셋 아티팩트를 아직 레이아웃 변경에 걸쳐 조정할
수 없으므로 마이그레이션이 거부됩니다. 프리셋을 제거하고, 마이그레이션한 다음,
다시 설치하세요.

`--skills`와 `--legacy-commands`는 함께 쓸 수 없습니다. 명시적인 선택자가 없으면,
관리되는 Spec Kit 스킬이 없는 한 관리되는 레거시 명령 파일이 기존 설치를 레거시
모드로 유지합니다. 관련 없는 `.bob/skills/` 디렉터리를 만드는 것만으로는
마이그레이션되지 않습니다. 레거시 명령 참조는 점 표기인 `/speckit.<command>`
형식을 유지합니다.

## 워크플로 권한

IDE 설정에는 필요 없지만, 워크플로 디스패치에는 추가로 `bob` CLI가 필요합니다.
위치 인자 프롬프트와 함께 `bob run --trust --accept-license`를 사용하며, 스트리밍
출력에는 `-f pretty`를, 캡처된 출력에는 `-f json`을 선택합니다.
trust 및 license 플래그는 이 비대화형 호출에 적용됩니다.

어댑터는 워크플로의 `model` 설정을 전달하지 않습니다. Bob은 자체 설정의
`session.model`을 통해 모델을 선택합니다.
