# 프리셋 게시하기

버전이 지정된 프리셋 아카이브를 게시하고, Spec Kit으로 테스트하고, Preset
Submission 이슈를 제출해 커뮤니티 카탈로그에서 탐색할 수 있게 하세요.
새 항목, 버전 업데이트, 수정에 모두 같은 이슈 흐름을 사용하세요.
손으로 편집한 카탈로그 풀 리퀘스트를 제출하지 마세요.

커뮤니티 등재는 코드 인증이 아닙니다. 메인테이너는 카탈로그 항목의 완전성과
형식을 확인할 뿐, 커뮤니티 프리셋 코드를 검토, 감사, 보증, 지원하지 않습니다.
사용자는 서드파티 프리셋을 설치하기 전에 검증해야 합니다.

## 패키지 준비

제출하기 전에, 동작하는 `preset.yml`, 프리셋 사용법 README, 오픈소스 `LICENSE`를
담은 공개 접근 가능한 저장소를 마련하세요.
`team-traceability`와 `1.0.0`처럼 설명적이고 문자로 시작하며 소문자와 하이픈으로
이루어진 ID와 숫자 `X.Y.Z` 버전을 사용하세요. 설명은 간결하게(200자 미만)
유지하고 유용한 소문자 태그를 두 개에서 다섯 개 고르세요.
변경 로그를 권장합니다.

개발 가이드에서 시작하고 매니페스트 필드와 의존성은 API 레퍼런스를 확인하세요.
선언된 모든 기여 파일을 포함하고, 명령이 유효 템플릿이 제공하는 섹션을
참조하는지 확인하세요. 로컬 매니페스트 검증은 보안 검토도, 완전한 게시
체크리스트도 아닙니다.

## 로컬 및 릴리스 설치 테스트

초기화된 테스트 프로젝트에서:

```bash
specify preset add --dev /path/to/team-traceability
specify preset info team-traceability
specify preset resolve spec-template
specify preset list
specify preset remove team-traceability
```

적어도 하나의 실제 프로젝트에서 테스트하세요. 명령 기여의 경우 지원되는 활성
통합으로 전달된 명령이나 스킬을 확인하고 호출해 보세요.
설치가 성공적으로 종료되는지만이 아니라 쌓기, 합성, 제거/복원을 확인하세요.

버전에 대한 태그와 GitHub 릴리스를 만드세요. 예:

```bash
git tag v1.0.0
git push origin v1.0.0
```

태그를 푸시하는 것만으로는 GitHub 릴리스가 만들어지지 않습니다. GitHub나 릴리스
도구를 통해 그 릴리스를 만드세요. 제출은 제출된 저장소에 속한 고정된 ZIP URL을
받습니다.

```text
https://github.com/<owner>/<repo>/archive/refs/tags/v1.0.0.zip
https://github.com/<owner>/<repo>/releases/download/v1.0.0/<asset>.zip
```

태그는 `vX.Y.Z`, `X.Y.Z`, 또는 `team-traceability-v1.0.0`처럼 일치하는 버전
접미사를 가진 범위를 사용할 수 있습니다. 유동적인 `releases/latest/` URL은
받아들여지지 않습니다. 그 태그에 대한 GitHub 릴리스가 존재해야 합니다.

CLI는 `.tar.gz`와 `.tgz`도 지원하지만, 커뮤니티 제출 계약은 현재 태그에 고정된
ZIP을 요구합니다. CLI 직접 설치의 경우 `preset.yml`이 아카이브 루트 또는 하나의
최상위 디렉터리 안에 있어야 합니다.
제출 검증기는 모노레포에서 프리셋 범위의 매니페스트를 찾을 수 있지만, 그렇다고
임의로 중첩된 모노레포 아카이브를 CLI로 설치할 수 있게 되지는 않습니다. 필요하면
프리셋 전용 자산을 게시하세요.

제출할 정확한 아카이브 URL을 테스트하세요.

```bash
specify preset add --from https://github.com/example/team-traceability/archive/refs/tags/v1.0.0.zip
specify preset resolve spec-template
specify preset remove team-traceability
```

## 사용법 README 요구 사항

제출의 Documentation URL은 더 넓은 프레임워크나 별도 CLI에 대한 홍보가 아니라 이
프리셋을 사용하는 방법을 설명하는, 읽을 수 있는 GitHub 호스팅 README를 가리켜야
합니다. 허용되는 URL 형식은 GitHub `blob` 또는 `raw` URL과
`raw.githubusercontent.com` URL입니다. 경로는 `README.md`(대소문자 구분 없음)로
끝나야 합니다. 쿼리 문자열과 프래그먼트는 가져올 때 무시됩니다.

모노레포에서는 저장소 개요 대신 `presets/team-traceability/README.md` 같은 프리셋
범위의 README를 우선하세요. 그 README가 아카이브 안에도 포함되는지와 상관없이,
릴리스를 다운로드하기 전에 접근할 수 있도록 유지하세요.

프리셋이 무엇을 제공하는지, 언제 사용하고 언제 사용하지 말아야 하는지, 지원되는
통합과 의존성, 그리고 유효한 설치 명령을 포함하세요. 다음을 우선합니다.

```bash
specify preset add --from https://github.com/example/team-traceability/archive/refs/tags/v1.0.0.zip
```

`--from` URL은 제출된 Download URL과 정확히 일치해야 합니다. 유효한
`specify preset add <id>` 또는 `specify preset add --dev <path>` 명령도 검증에서
받아들여집니다. ID 설치는 여전히 설치 허용 카탈로그가 필요하므로, 커뮤니티
사용자에게는 고정된 아카이브를 직접 지정하는 것이 가장 명확합니다.

유효한 설치 명령이 없는 README는 검증에 실패합니다. 이 프리셋에 속한 것으로 식별할
수 있는 오래된 릴리스 URL도 다른 유효한 명령이 있더라도 실패합니다. 모노레포에
있는 관련 없는 프리셋의 릴리스 예시는 자동으로 오래된 것으로 취급되지 않고
구분됩니다.

## 커뮤니티 카탈로그에 제출하기

[Preset Submission 이슈](https://github.com/github/spec-kit/issues/new?template=preset_submission.yml)를
여세요. ID, 이름, 버전, 설명, 작성자, 저장소 URL, 고정된 Download URL,
Documentation URL, 라이선스, 필요한 Spec Kit 버전, 필요한 확장 ID, 제공하는
템플릿/명령/스크립트, 태그, 주요 기능을 제공하세요. 테스트 및 제출 체크리스트를
완료하세요.

필요한 확장은 `preset.yml`에도 선언하세요. 카탈로그 메타데이터만으로는 로컬 및
직접 아카이브 설치에 의존성을 전달할 수 없습니다.

선택적인 제출 체크섬은 정확히 64개의 16진수 문자와 함께 `### SHA-256` 제목을
이슈에 덧붙여 제공할 수 있습니다. 양식에 전용 체크섬 필드는 없습니다. 제공하면
가져온 아카이브와 일치해야 합니다. 불일치는 검증 실패이며, 자동화가 조용히
교체하는 값이 아닙니다.

제출을 대신해 `presets/catalog.community.json`이나 `docs/community/presets.md`를
직접 편집하지 마세요.
자동화가 검증 후에 그 변경을 준비합니다.

## 제출 이후에 일어나는 일

1. 이슈 양식이 `triage-must-have` 접수 판정을 적용합니다. 이것으로 카탈로그
   검증이 시작되지는 않습니다.
2. 분류 중에 메인테이너가 검증 워크플로를 시작하기 위해 별도로
   `preset-submission`을 적용합니다. 기여자는 그 레이블을 적용하거나 요청할
   필요가 없습니다.
3. 검증은 제출 메타데이터, 저장소/라이선스 접근 가능 여부, 링크된 README와 설치
   참조, 릴리스 고정, 아카이브 접근 가능 여부, 선택적인 제출 체크섬, 필수
   체크리스트를 확인합니다. 검증기는 아카이브 콘텐츠를 실행하지 않고 게시된
   매니페스트를 파싱해, 그 ID, 버전, Spec Kit 요구 사항, 필요한 확장을 이슈와
   비교합니다.
4. 성공하면 자동화가 카탈로그와 커뮤니티 표를 갱신하는 초안 풀 리퀘스트를
   제안합니다. 실제 아카이브 다이제스트를 기록하고, 업데이트 시 `created_at`을
   보존하고, 카탈로그 ID와 표시 이름을 적절히 정렬하고, 검증 통과로 표시하기 전에
   생성된 파일을 검증합니다.
5. 메인테이너가 생성된 카탈로그 변경을 검토하고 승인되면 병합합니다. 프리셋은
   탐색할 수 있게 되며, 커뮤니티 카탈로그는 탐색 전용으로 남습니다.

확인된 제출 결함은 `validation-failed`와 함께 보고됩니다.
권한, 네트워크, 도구 관련 차단 요인은 성공한 검사나 반드시 프리셋의 결함으로
보고되는 것이 아니라 차단됨으로 보고됩니다. 워크플로는 실행할 수 없었던 것을
보고해 메인테이너가 조사하고 다시 실행할 수 있게 합니다.

이는 메타데이터와 아티팩트 일관성 검사이며, 프리셋 지침, 템플릿 품질, 실행 코드의
안전성에 대한 검토가 아닙니다. 커뮤니티 코드 승인이나 지원 보장은 없습니다.

## 릴리스 업데이트

매니페스트 버전과 변경 로그를 갱신하고, 변경할 수 없는 새 태그 릴리스를 게시하고,
정확한 아카이브를 다시 테스트하세요. 그 릴리스에 맞게 README의 설치 참조를 갱신한
다음, 새 버전과 URL로 또 하나의 Preset Submission 이슈를 제출하세요. 깨진 항목을
수정할 때도 같은 이슈 흐름이 적용됩니다.

직접 운영하는 비공개 카탈로그라면 카탈로그 레퍼런스에 따라 현재 및 과거 릴리스
기록을 유지 관리하세요. 커뮤니티 제출 절차가 릴리스 이력을 게시하거나 그
카탈로그를 설치 허용으로 만든다고 가정하지 마세요.

## 함께 보기

- [프리셋 개요](overview.md)
- [프리셋 개발하기](developing.md)
- [매니페스트 API](api.md)
- [카탈로그 항목과 과거 릴리스](catalogs.md)
- [CLI 레퍼런스](cli.md)
- [커뮤니티 프리셋](../community/presets.md)
