# 번들 개발하기

번들은 매니페스트와 사용 문서를 통해 기존 컴포넌트 스택을 배포합니다. 역할이나
팀이 얻을 결과에서 시작해, 그것을 제공하는 컴포넌트를 선택하고, 결합된 설치를
테스트하세요. 번들은 확장, 프리셋, 워크플로, 스텝 작성을 대체하지 않습니다.

## 번들 디렉터리 만들기

매니페스트와 README를 디렉터리 루트에 두세요.

```text
team-setup/
  bundle.yml
  README.md
  LICENSE
```

이 최소 예시는 패키지된 bug 확장과 워크플로를 사용합니다.

```yaml
schema_version: "1.0"
bundle:
  id: team-setup
  name: Team setup
  version: 1.0.0
  role: developer
  description: A bug-fixing setup for the team
  author: Example team
  license: MIT
requires:
  speckit_version: ">=0.9.0"
provides:
  extensions:
    - id: bug
      version: 1.0.2
  workflows:
    - id: bugfix
      version: 1.0.0
tags: [developer, bugfix]
```

고정할 버전은 테스트한 릴리스에서 고르세요. 호환되는 더 새로운 안정 패키지 패치가
경고와 함께 이 고정을 충족할 수 있습니다. 명시적인 컴포넌트 카탈로그 소스는 정확한
고정 적용을 유지합니다.

README는 의도한 역할, 설치되는 컴포넌트, 통합 요구 사항, 사용자가 등록해야 하는
카탈로그, 설치 명령, 설치된 구성을 사용하는 방법을 설명해야 합니다. 커뮤니티에
게시하려면 라이선스 파일을 포함하세요.

## 컴포넌트와 그 소스 선택하기

확장, 프리셋, 워크플로 참조에는 정확한 버전이 필요합니다. 스텝은 버전을 생략할 수
있으며, 그 경우 설치는 현재 스텝 카탈로그 릴리스를 선택합니다. 재현성이 중요하다면
스텝도 고정하세요.

프리셋은 최소 `1`인 정수 우선순위와 합성 전략을 선언해야 합니다. 확장도 우선순위를
선언할 수 있으며 기본값은 `10`입니다.
전체 필드 계약은 API 레퍼런스를 사용하세요.

선택적인 컴포넌트 `source`는 이길 것으로 기대하는 컴포넌트 카탈로그의 이름을
지정합니다. URL도, 디렉터리도, 카탈로그 우선순위를 바꾸는 메커니즘도 아닙니다.
소스를 선택하기 전에 유효 카탈로그를 확인하세요.

```bash
specify extension catalog list
specify preset catalog list
specify workflow catalog list
specify workflow step catalog list
```

번들에 필요한 기본이 아닌 모든 카탈로그를 해당 `catalog add` 명령과 함께
문서화하세요. 카탈로그는 분리된 채로 남습니다. 번들 카탈로그를 추가한다고 해서
컴포넌트 카탈로그가 추가되지는 않습니다. 사용자는 설치를 허용하기 전에 이를
검증해야 합니다. 컴포넌트 카탈로그 이름은 고유하게 사용하세요. 여러 활성 카탈로그와
일치하는 명시적 소스는 모호하므로 거부됩니다.

로컬 번들 자산은 배포를 위해 패키징되지만 컴포넌트 페이로드로 자동 설치되지는
않습니다. 특히 사용자 정의 스텝을 `steps/<id>/` 아래에 복사한다고 해서 해석
가능해지지 않습니다. 접근 가능한 스텝 카탈로그를 통해 게시하고 그 의존성을
문서화하세요.

## 기존 번들 합성하기

스키마 `1.1`과 `provides.bundles`를 사용해 다른 번들의 정확한 릴리스를 참조하세요.
예를 들어 카탈로그에서 해당 릴리스를 사용할 수 있는지 확인한 뒤:

```yaml
schema_version: "1.1"
bundle:
  id: team-setup
  name: Team setup
  version: 1.0.0
  role: developer
  description: A composed team setup
  author: Example team
  license: MIT
requires:
  speckit_version: ">=0.9.0"
provides:
  bundles:
    - id: bugfix
      version: 1.0.2
      source: default
```

참조는 상위가 로컬 매니페스트일 때를 포함해 유효 번들 카탈로그를 통해 해석됩니다.
로컬 경로 참조가 아니며 `build`가 내장하지도 않습니다. 상위 번들 자신의 컴포넌트
선언은 참조된 선언을 재정의할 수 있습니다. 호환되지 않는 형제 선언에는 상위의
명시적인 해결이 필요합니다. API 레퍼런스가 순서, 중복 제거, 한계를 정의합니다.

퍼스트파티 번들 카탈로그는 현재 과거 릴리스를 유지하지 않습니다. 정확한 참조는
카탈로그가 다음 릴리스로 넘어가면 해석되지 않을 수 있습니다. 오래 유지되는
합성에는 필요한 이력을 보존하는 카탈로그를 사용하세요.

## 검증과 빌드

번들을 담은 저장소나 디렉터리에서:

```bash
specify bundle validate --path ./team-setup
specify bundle build --path ./team-setup --output ./dist
```

오류뿐 아니라 경고도 읽으세요. 온라인 검증은 컴포넌트 해석을 확인하고 참조를
확장하지만, 실행 중인 CLI의 Spec Kit 버전 게이트를 강제하지는 않습니다. 오프라인
검증은 네트워크 전용 컴포넌트를 확인하거나 번들 참조를 확장할 수 없습니다.

빌드는 일반 `README.md`와 `bundle.yml`이 필요하고 매니페스트 구조를 확인합니다.
참조가 해석되는지나 설치가 성공하는지는 확인하지 않습니다. 결과 ZIP에는 루트 수준
파일과 로컬 자산이 들어 있으며, 다운로드된 컴포넌트 페이로드나 참조된 번들은 들어
있지 않습니다.

빌드 출력은 동일한 입력에 대해 재현 가능합니다. 파일은 POSIX 아카이브 경로로
정렬되고, 타임스탬프는 고정되며, 권한은 실행 가능 또는 실행 불가 모드로
정규화됩니다. 심볼릭 링크, `.git`, `__pycache__`, `.DS_Store`, 이 번들의 이전 버전
빌드 아티팩트, 중첩된 출력 디렉터리는 제외됩니다.

## 설치와 변경 테스트하기

의도한 통합을 갖춘 깨끗한 프로젝트를 사용하세요. 검증한 의존성 카탈로그를 등록한
다음, 사용자가 받게 될 정확한 아티팩트를 설치합니다.

```bash
specify init --here --integration copilot
specify bundle install /path/to/dist/team-setup-1.0.0.zip
specify bundle list --json
```

전달된 명령을 확인하고 설치된 워크플로나 그 밖의 의도한 기능을 실행해 보세요.
설치를 반복해 변경되지 않은 컴포넌트와 로컬 편집이 보존되는지 확인하세요.
독립적인 컴포넌트나 다른 번들의 컴포넌트가 설치된 상태로 남아 있을 때의 제거를
테스트하세요.

로컬 개발에서 일반 재설치는 업데이트 작업이 아닙니다. 수정된 버전, 변경된 소유
컴포넌트 설정, 제거된 컴포넌트는 다음으로 적용하세요.

```bash
specify bundle install /path/to/team-setup/bundle.yml --refresh
```

새로 고침은 업데이트 경로를 통해 소유한 컴포넌트를 덮어쓸 수 있으며 실패 시 완전히
롤백되지 않습니다. 이 테스트에는 일회용 프로젝트를 사용하세요.
유효한 설치와, 사용할 수 없는 고정, 통합 불일치, 다른 번들의 요구 사항과의 충돌
같은 관련 거부 사례를 모두 테스트하세요.

카탈로그를 통해 배포한다면 그 설치 허용 카탈로그에서의 ID 설치도 테스트하세요.
아티팩트 직접 설치가 성공했다고 해서 카탈로그 우선순위, 다운로드 메타데이터, 과거
릴리스 선택이 검증되지는 않습니다.

## 함께 보기

- [번들 개요](overview.md)
- [매니페스트 및 번들 참조 API](api.md)
- [CLI 레퍼런스](cli.md)
- [카탈로그 설정](catalogs.md)
- [번들 게시하기](publishing.md)
