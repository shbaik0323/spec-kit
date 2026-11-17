# 번들 게시하기

버전이 지정된 번들 아티팩트를 게시하고, 그 배포 경로를 테스트한 다음, Bundle
Submission 이슈를 통해 메타데이터를 제출하세요. 새 항목, 버전 업데이트, 수정에
모두 같은 이슈 흐름을 사용하세요. 제출을 대신해 커뮤니티 카탈로그나 그 문서 표를
손으로 편집하지 마세요.

커뮤니티 게시는 코드 인증이 아닙니다. 메인테이너는 메타데이터, 링크, 아티팩트
일관성, 제출된 설치 근거를 확인할 뿐, 번들 코드나 그것이 설치하는 컴포넌트를
검토, 감사, 보증, 지원하지 않습니다. 사용자는 설치 전에 그 컴포넌트를 검증해야
합니다.

## 릴리스 준비

`bundle.yml`, `README.md`, 라이선스 파일(`LICENSE`, `LICENSE.md`, 또는
`LICENSE.txt`)을 담은 공개 GitHub 저장소를 제공하세요. README는 의도한 역할,
설치되는 컴포넌트, 필요한 카탈로그, 설치 및 사용 단계를 설명해야 합니다.

매니페스트 계약에 맞는 안전한 번들 ID를 사용하세요. 커뮤니티 제출 버전은 `v`
접두사 없는 숫자 `X.Y.Z`여야 합니다. 이는 매니페스트 파서의 일반적인 SemVer
지원보다 좁습니다. 소문자 태그를 두 개에서 다섯 개 고르세요.

온라인으로 검증하고 버전이 지정된 아티팩트를 빌드합니다.

```bash
specify bundle validate --path ./team-setup
specify bundle build --path ./team-setup --output ./dist
```

경고를 해결하고 의도한 컴포넌트 릴리스를 사용할 수 있는지 확인하세요. 빌드는 구조
검사만 수행합니다. 빌드된 ZIP은 설치 근거가 아닙니다. 매니페스트와 README는 심볼릭
링크가 아니라 일반 파일로 포함하세요.

태그와 GitHub 릴리스를 만든 다음, 생성된 `team-setup-1.0.0.zip`을 첨부하세요.
커뮤니티 Download URL은 제출된 저장소에 속해야 하며 다음 형식을 따라야 합니다.

```text
https://github.com/<owner>/<repo>/releases/download/<tag>/<id>-<version>.zip
```

태그는 제출된 버전과 일치해야 합니다. `v1.0.0`, `1.0.0`, 또는
`team-setup-v1.0.0`처럼 일치하는 접미사를 가진 범위 지정 태그입니다. 유동적인
`releases/latest/` URL과 소스 아카이브는 커뮤니티 번들 자산 계약이 아닙니다.
릴리스와 첨부된 정확한 ZIP 자산이 존재해야 합니다.

## 배포 경로 테스트

의도한 통합을 갖추고 필요한 모든 검증된 컴포넌트 카탈로그가 등록된 깨끗한 Spec
Kit 프로젝트에서 테스트하세요. 릴리스 아티팩트 설치는 작성자의 프로젝트에 남아
있던 컴포넌트에 의존하지 않고 동작해야 합니다.

예를 들어 소스를 검증한 뒤:

```bash
specify init --here --integration copilot
specify preset catalog add https://example.com/presets.json \
  --name team --install-allowed
specify extension catalog add https://example.com/extensions.json \
  --name team --install-allowed
curl -L -o team-setup-1.0.0.zip \
  https://github.com/example/team-setup/releases/download/v1.0.0/team-setup-1.0.0.zip
specify bundle install ./team-setup-1.0.0.zip
specify bundle list --json
```

해당하는 경우 워크플로 및 스텝 카탈로그를 포함해, 번들에 실제로 필요한 카탈로그
명령만 사용하세요. 모든 컴포넌트가 패키지된 소스나 기본 소스로부터 해석된다면
필요한 컴포넌트 카탈로그에 `None`이라고 보고하세요.

번들 카탈로그 항목을 제안한다면, 별도의 깨끗한 프로젝트에서 설치 허용
카탈로그로부터의 ID 설치를 테스트하세요.

```bash
specify bundle catalog add https://example.com/bundles.json \
  --id team --policy install-allowed
specify bundle install team-setup
```

그 프로젝트에도 필요한 컴포넌트 카탈로그를 등록하세요. 번들 카탈로그는
매니페스트의 위치를 알려 줄 뿐, 동반 카탈로그를 등록하지 않습니다. 내장 커뮤니티
카탈로그는 게시 후에도 탐색 전용으로 남습니다.

검증, 빌드, 아티팩트 설치, 해당하는 경우 카탈로그 ID 설치, 설치된 명령/워크플로의
실제 사용, 제거를 기술하세요. 제출하는 테스트 세부 사항에는 테스트한 Spec Kit 버전,
운영 체제, 통합, 의존성 카탈로그를 포함하세요.

## 메타데이터 제출

[Bundle Submission 이슈](https://github.com/github/spec-kit/issues/new?template=bundle_submission.yml)를
여세요. ID, 이름, 버전, 역할/팀, 설명, 작성자, 저장소 URL, Download URL,
Documentation URL, 라이선스, 필요한 Spec Kit 버전, 선택적인 통합 대상, 컴포넌트
요약, 필요한 카탈로그, 태그, 주요 기능, 테스트 세부 사항, 사용 예시, 제안하는
카탈로그 항목을 제공하세요.

필수 테스트 및 제출 체크박스를 모두 완료하세요. 매니페스트, README, 이슈
메타데이터, 카탈로그 제안, 테스트 근거가 서로 일치해야 합니다. 필요한 동반
카탈로그에는 HTTPS URL과 문서화된 설정 명령이 있어야 합니다. 제출 자동화는 이를
등록하지 않습니다.

선택적인 제출 체크섬은 제안하는 카탈로그 항목의 `sha256`으로, 또는 덧붙인
`### SHA-256` 제목 아래에 제공할 수 있습니다. 정확히 64개의 16진수 문자를 담아야
합니다. 두 위치에 모두 제공하면 공백을 제거하고 대소문자를 정규화한 뒤 일치해야
합니다. 형식이 잘못되었거나 일치하지 않는 체크섬은 검증을 실패시킵니다. 조용히
교체되지 않습니다.

## 제출 이후에 일어나는 일

1. 이슈 양식이 `triage-must-have` 접수 판정을 적용합니다. 이것으로 카탈로그
   검증이 시작되지는 않습니다.
2. 메인테이너가 분류 중에 이슈를 검토하고, 자동화된 워크플로를 시작하기 위해
   별도로 `bundle-submission`을 적용합니다. 기여자는 그 레이블을 적용하거나
   요청할 필요가 없습니다.
3. 자동화가 필드, 저장소/문서 접근 가능 여부, 매니페스트 일관성, 라이선스 유무,
   릴리스 고정, 실제 아티팩트 다운로드와 선택적 체크섬, 카탈로그 형태, 필요한
   카탈로그, 테스트 확인 사항을 검증합니다. 다운로드한 콘텐츠를 실행하거나 코드를
   감사하지 않습니다.
4. 성공하면 자동화가 `bundles/catalog.community.json`과
   `docs/community/bundles.md`를 갱신하는 초안 풀 리퀘스트를 제안합니다. 실제로
   다운로드한 아카이브의 SHA-256을 기록하고 `verified: false`를 유지합니다.
5. 메인테이너가 카탈로그 변경을 검토하고 승인되면 병합합니다.
   항목은 `specify bundle search`로 탐색할 수 있게 됩니다.

확인된 제출 결함은 수정 세부 사항과 함께 `validation-failed`를 받습니다. 권한,
네트워크, 도구 관련 차단 요인은 통과한 검사나 반드시 번들의 결함으로 보고되는 것이
아니라 차단됨으로 보고됩니다. 체크섬 불일치나 관찰된 아티팩트 누락은 제출
실패입니다.

## 업데이트와 수정

변경할 수 없는 새 태그 릴리스를 게시하고, 매니페스트와 README의 참조를 갱신하고,
정확한 아티팩트와 해당하는 카탈로그 설치 경로를 다시 테스트하세요. 또 하나의
Bundle Submission 이슈를 제출하고 업데이트하는 기존 항목을 밝히세요.
이슈에서 같은 버전에서의 메타데이터 전용 수정임을 명시적으로 설명하지 않는 한
버전은 올라가야 합니다.

직접 운영하는 비공개 카탈로그라면 현재 및 과거 릴리스 기록에 대해 카탈로그
레퍼런스를 따르세요. 커뮤니티 제출 흐름이 이전 릴리스를 유지하거나 내장 소스를
설치 허용으로 바꾼다고 가정하지 마세요.

## 함께 보기

- [번들 개발 및 테스트](developing.md)
- [매니페스트 API](api.md)
- [카탈로그 항목과 릴리스 이력](catalogs.md)
- [CLI 레퍼런스](cli.md)
- [커뮤니티 번들 목록](../community/bundles.md)
- [번들 개요](overview.md)
