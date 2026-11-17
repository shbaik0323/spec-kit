# 프리셋 개발하기

프리셋은 사용자 정의 문서 템플릿, 에이전트 지침, 스크립트를 `preset.yml`
매니페스트가 있는 디렉터리에 패키징합니다. 출력 구조만 바뀐다면 템플릿 기여를
우선하고, 프로세스 자체에 다른 지침이 필요하다면 명령을 오버라이드하세요. 일회성
프로젝트 변경이라면 배포 가능한 프리셋보다 프로젝트 로컬 오버라이드가 더 간단할 수
있습니다.

## 스캐폴드에서 시작하기

저장소의
[프리셋 스캐폴드](https://github.com/github/spec-kit/tree/main/presets/scaffold)를
여러분이 관리하는 디렉터리로 복사하세요.

```bash
cp -r presets/scaffold my-preset
```

ID, 이름, 버전, 설명, 필요한 Spec Kit 버전, 기여를 편집하세요. 예시 확장 명령
오버라이드를 포함해 사용하지 않는 예시는 실수로 배포하지 않도록 제거하세요.

일반적인 패키지는 다음을 담습니다.

```text
my-preset/
  preset.yml
  README.md
  LICENSE
  CHANGELOG.md
  .presetignore
  templates/
    spec-addendum.md
  commands/
    speckit.specify.md
  scripts/
    my-helper.sh
```

프리셋이 사용하는 디렉터리만 포함하세요. 매니페스트의 `provides.templates` 목록은
문서 템플릿만이 아니라 세 가지 유형의 기여를 모두 담습니다. `file` 필드는 관례와
다른 경로를 가리킬 수 있습니다. 사용자 정의하는 논리적 아티팩트는 `name`으로
식별하세요.

## 기여 선언하기

예를 들어 유효 명세 템플릿에 추적성 섹션을 덧붙이려면 다음과 같이 합니다.

```yaml
schema_version: "1.0"

preset:
  id: "team-traceability"
  name: "Team Traceability"
  version: "1.0.0"
  description: "Adds team traceability requirements to specifications"
  author: "Example Team"
  repository: "https://github.com/example/team-traceability"
  license: "MIT"

requires:
  speckit_version: ">=0.9.0"

provides:
  templates:
    - type: "template"
      name: "spec-template"
      file: "templates/spec-addendum.md"
      description: "Traceability section"
      strategy: "append"

tags:
  - "traceability"
  - "governance"
```

제목은 명확하게, 자리 표시자는 명시적으로 유지하세요. 명령이 템플릿 섹션을
참조한다면 유효 템플릿에 실제로 그 섹션이 있는지 확인하세요.
프리셋이 다른 프리셋과 합성되도록 필요한 기여만 선언하세요.

`wrap`의 경우 콘텐츠 파일에 템플릿과 명령은 `{CORE_TEMPLATE}`, 스크립트는
`$CORE_SCRIPT`가 들어 있어야 합니다. 이 자리 표시자는 해석된 더 낮은 우선순위
콘텐츠를 뜻하며, 반드시 수정되지 않은 코어 콘텐츠를 뜻하지는 않습니다.
append, prepend, wrap 기여에는 하위의 `replace` 기반이 필요합니다. 자기 완결적인
새 명령은 보통 `replace`를 사용합니다.

프리셋이 확장 기능에 의존한다면 `preset.yml`에 확장 의존성을 선언하세요.
네임스페이스만으로는 의존성이 성립하거나 등록이 억제되지 않습니다. 경고 동작과
선택적/버전 제약 의존성은 API 레퍼런스를 참고하세요.

<a id="exclude-development-files-with-presetignore"></a>

## `.presetignore`로 개발 파일 제외하기

소스 루트의 `preset.yml` 옆에 바이트 순서 표시가 있든 없든 UTF-8 `.presetignore`를
두세요.

```gitignore
.git/
.idea/
tests/
```

필터링은 `preset update`로 설치되는 대체본을 포함해, 복사되는 로컬 디렉터리와
추출되는 아카이브에 적용됩니다. 이 파일이 없으면 이전처럼 모든 소스 파일이
복사됩니다. `.gitignore`는 설치 필터링에 사용되지 않으며, 암묵적인 개발 파일
제외는 없습니다.

패턴은 `.extensionignore`와 마찬가지로 Git 방식의 매칭과 정규화를 사용합니다.

- 한 줄에 패턴 하나를 사용합니다. 빈 줄과 `#` 주석은 무시됩니다.
- `*`와 `?`는 디렉터리 경계를 넘지 않습니다. `**`는 넘을 수 있습니다.
- 끝의 `/`는 디렉터리만 매칭합니다. 맨 앞의 `/`, 또는 끝이 아닌 다른 곳의 `/`는
  패턴을 소스 루트에 고정합니다. 그런 슬래시가 없는 패턴은 어느 깊이의 이름과도
  매칭됩니다.
- 뒤에 오는 `!` 패턴은 매칭을 다시 포함시킵니다. 제외된 디렉터리는 순회되지
  않으므로, 상위가 제외된 동안에는 하위를 다시 포함시킬 수 없습니다.
- 공백은 제거되고 백슬래시는 `/`로 정규화됩니다. 비ASCII 이름에는 UTF-8을
  사용하세요.

`docs/` 안에서 `docs/api.md`만 남기려면 디렉터리가 아니라 그 내용을 제외하세요.

```gitignore
docs/*
!docs/api.md
```

`docs/`에 이어 `!docs/`와 `!docs/api.md`를 쓰면, 다른 규칙이 제외하지 않는 한
`docs/` 아래의 모든 파일이 다시 포함됩니다.

소스 루트의 `.presetignore`만 규칙을 추가합니다. 중첩된 ignore 파일은 평가되지
않습니다. 루트 ignore 파일이 있으면 어느 깊이에 있든 `.presetignore`라는 이름의
파일은 부정 패턴이 있어도 제외됩니다. 루트 ignore 파일이 없으면 중첩된
`.presetignore` 파일은 변경 없이 복사됩니다.

매니페스트와 선언된 자산 파일도 예외가 아닙니다. `preset.yml`과 필요한 모든 기여
파일이 포함되도록 유지하세요. 이를 걸러 내면 불완전하거나 사용할 수 없는 설치가
만들어질 수 있습니다.

디렉터리 설치 프로그램은 Python 관리자 API를 통한 강제 교체를 포함해, 기존 대상을
제거하기 전에 ignore 규칙을 검증합니다. 유효하지 않은 UTF-8과 형식이 잘못된
패턴은 프리셋 검증 오류를 발생시킵니다.
CLI의 별도 `update` 명령은 여전히 먼저 제거한 다음 add를 호출합니다. 대체본의
ignore 파일을 사전 점검하지 않습니다.

## 로컬에서 테스트하기

프리셋이 지원하는 통합과 명령/스킬 모드를 갖춘 초기화된 테스트 프로젝트를
사용하세요. 그 프로젝트에서:

```bash
specify preset add --dev /path/to/my-preset --priority 5
specify preset info team-traceability
specify preset list
specify preset resolve spec-template
specify artifact info template:spec-template --json
```

기여가 설치된 디렉터리에만이 아니라 생성된 아티팩트에 있는지 확인하세요. 명령
오버라이드의 경우 활성 통합에 전달된 명령이나 스킬을 확인하고 그 에이전트로
호출해 보세요. 모든 통합이 같은 파일 레이아웃을 사용한다고 가정하지 말고 관련
형식과 모드를 테스트하세요.

서로 다른 우선순위와 같은 우선순위에서 겹치는 프리셋, 프로젝트 로컬 오버라이드,
제거/복원, 누락된 의존성이나 합성 기반을 테스트하세요. 실행 가능한 스크립트를
배포한다면 의도한 동작과 실패를 테스트하세요. PowerShell 스크립트는 Windows
PowerShell 5.1을 대상으로 해야 합니다. 활성화, 비활성화, 우선순위 변경만으로 이미
전달된 명령이나 스킬 파일이 다시 쓰일 것이라고 기대하지 마세요.

개발 소스를 편집한 뒤에는 새 사본을 설치하세요. `--dev`는 소스를 감시하거나
링크하지 않습니다.

```bash
specify preset remove team-traceability
specify preset add --dev /path/to/my-preset --priority 5
```

대안인 `update --dev`는 의도적으로 제거 후 추가 방식으로 동작하며, 새 소스가
유효하지 않으면 프리셋이 제거된 채로 남을 수 있습니다. 게시하기 전에, 특히
패키징이 디렉터리 레이아웃을 바꾼다면 아카이브 설치도 테스트하세요.

## 함께 보기

- [프리셋 개요](overview.md)
- [CLI 레퍼런스](cli.md)
- [매니페스트 및 합성 API](api.md)
- [카탈로그](catalogs.md)
- [프리셋 게시하기](publishing.md)
