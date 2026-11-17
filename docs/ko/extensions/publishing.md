# 확장 게시 가이드

이 가이드는 확장을 Spec Kit 확장 카탈로그에 게시해 `specify extension search`로 탐색할 수 있게 하는 방법을 설명합니다.

## 목차

1. [사전 요구 사항](#prerequisites)
2. [확장 준비하기](#prepare-your-extension)
3. [카탈로그에 제출하기](#submit-to-catalog)
4. [릴리스 워크플로](#release-workflow)
5. [모범 사례](#best-practices)

---

<a id="prerequisites"></a>

## 사전 요구 사항

확장을 게시하기 전에 다음을 갖추었는지 확인하세요.

1. **유효한 확장**: 유효한 `extension.yml` 매니페스트를 갖춘 동작하는 확장
2. **GitHub 저장소**: 커뮤니티 카탈로그 제출을 위해 공개 GitHub 저장소에 호스팅된
   확장
3. **문서**: 설치 및 사용 지침이 담긴 README.md
4. **라이선스**: 오픈소스 라이선스 파일(MIT, Apache 2.0 등)
5. **버전 관리**: 시맨틱 버저닝(예: 1.0.0)
6. **테스트**: 실제 프로젝트에서 테스트한 확장

---

<a id="prepare-your-extension"></a>

## 확장 준비하기

### 1. 확장 구조

확장이 표준 구조를 따르는지 확인하세요.

```text
your-extension/
├── extension.yml              # Required: Extension manifest
├── README.md                  # Required: Documentation
├── LICENSE                    # Required: License file
├── CHANGELOG.md               # Recommended: Version history
├── .gitignore                 # Recommended: Git ignore rules
│
├── commands/                  # Extension commands
│   ├── command1.md
│   └── command2.md
│
├── config-template.yml        # Config template (if needed)
│
└── docs/                      # Additional documentation
    ├── usage.md
    └── examples/
```

### 2. extension.yml 검증

매니페스트가 유효한지 확인하세요.

```yaml
schema_version: "1.0"

extension:
  id: "your-extension"           # Unique lowercase-hyphenated ID
  name: "Your Extension Name"     # Human-readable name
  version: "1.0.0"                # Semantic version
  description: "Brief description (one sentence)"
  author: "Your Name or Organization"
  repository: "https://github.com/your-org/spec-kit-your-extension"
  license: "MIT"
  homepage: "https://github.com/your-org/spec-kit-your-extension"

requires:
  speckit_version: ">=0.1.0"    # Required spec-kit version

provides:
  commands:                       # List all commands
    - name: "speckit.your-extension.command"
      file: "commands/command.md"
      description: "Command description"

tags:                             # 2-5 relevant tags
  - "category"
  - "tool-name"
```

**검증 체크리스트**:

- ✅ `id`가 소문자로 시작하고 그 밖에는 소문자, 숫자, 하이픈만 포함(커뮤니티
  제출의 경우 `^[a-z][a-z0-9-]*$`)
- ✅ `version`이 시맨틱 버저닝(X.Y.Z)을 따름
- ✅ `description`이 간결함(커뮤니티 제출의 경우 200자 미만)
- ✅ `repository` URL이 유효하고 공개되어 있음
- ✅ 모든 명령 파일이 확장 디렉터리에 존재함
- ✅ 태그가 소문자이고 설명적임

### 3. GitHub 릴리스 만들기

확장 버전에 대한 GitHub 릴리스를 만드세요.

```bash
# 릴리스에 태그 지정
git tag v1.0.0
git push origin v1.0.0

# GitHub에서 릴리스 만들기
# 이동: https://github.com/your-org/spec-kit-your-extension/releases/new
# - Tag: v1.0.0
# - Title: v1.0.0 - Release Name
# - Description: Changelog/release notes
```

릴리스 아카이브 URL은 다음과 같습니다.

```text
https://github.com/your-org/spec-kit-your-extension/archive/refs/tags/v1.0.0.zip
```

### 4. 설치 테스트

사용자가 릴리스에서 설치할 수 있는지 테스트하세요.

```bash
# dev 설치 테스트
specify extension add --dev /path/to/your-extension

# GitHub 아카이브에서 테스트
specify extension add <extension-name> --from https://github.com/your-org/spec-kit-your-extension/archive/refs/tags/v1.0.0.zip
```

---

<a id="submit-to-catalog"></a>

## 카탈로그에 제출하기

### 카탈로그 이해하기

Spec Kit은 내장 설치 소스와 탐색 소스를 가진 카탈로그 스택을 사용합니다.
설정, 릴리스 메타데이터, 신뢰 경계는 [확장 카탈로그](catalogs.md)를 참고하세요.

**확장 게시의 경우**: 모든 커뮤니티 확장은 `extensions/catalog.community.json`에
등재됩니다. 사용자는 릴리스를 직접 설치하거나 그 항목을 자신의 신뢰 카탈로그로
복사하기 전에 검증해야 합니다.

### 제출 방법

확장을 커뮤니티 카탈로그에 제출하려면 **[Extension Submission](https://github.com/github/spec-kit/issues/new?template=extension_submission.yml)** 템플릿으로 새 이슈를 제출하세요. 템플릿은 다음을 포함해 필요한 모든 메타데이터를 수집합니다.

- 확장 ID, 이름, 버전
- 설명, 작성자, 라이선스
- 저장소, 다운로드 URL, 문서 링크
- 필요한 Spec Kit 버전과 도구 의존성
- 명령과 훅의 수
- 태그와 주요 기능
- 테스트 확인

> [!IMPORTANT]
> `extensions/catalog.community.json`을 직접 편집하는 풀 리퀘스트를 열지 **마세요**. 모든 커뮤니티 확장 제출은 이슈 템플릿을 거쳐야 합니다. 이슈 템플릿을 사용하지 않은 제출은 닫히며, 템플릿을 사용해 다시 제출하라는 요청을 받게 됩니다.

<a id="submission-validation"></a>

### 제출 검증

커뮤니티 제출은 일반적인 확장 런타임보다 엄격합니다.

- 확장 ID는 `^[a-z][a-z0-9-]*$`와 일치해야 합니다.
- 버전은 숫자만으로 이루어진 `X.Y.Z`여야 하며 `v` 접두사가 없어야 합니다. 릴리스
  태그에는 `v` 접두사나 범위가 포함될 수 있지만, 그 버전은 제출된 버전과 일치해야
  합니다.
- 저장소는 공개여야 하며 `extension.yml`, `README.md`, `LICENSE`를 담고 있어야
  합니다.
- 다운로드 URL은 같은 GitHub 저장소에 속해야 하며
  `https://github.com/<owner>/<repo>/archive/refs/tags/<tag>.zip` 또는
  `https://github.com/<owner>/<repo>/releases/download/<tag>/<asset>.zip`을
  사용해야 합니다. 유동적인 `releases/latest/` URL은 거부됩니다.
- 아카이브에 접근할 수 있어야 합니다. SHA-256 다이제스트가 제출되면 다운로드한
  아카이브와 일치해야 합니다. 자동화는 제출되지 않았더라도 다운로드한 아카이브의
  다이제스트를 제안 카탈로그 항목에 기록합니다.

이는 메타데이터와 아카이브 출처 검사이며, 확장 코드에 대한 검토가 아닙니다. 다른
공개 호스팅과 비공개 릴리스는 여러분이 통제하는 카탈로그에서 사용할 수 있지만, 이
커뮤니티 제출 절차를 통해서는 받아들여지지 않습니다.

### 제출 이후에 일어나는 일

1. Extension Submission 양식으로 이슈가 열리면 GitHub가 `triage-must-have` 판정을 적용합니다. 이 접수 자동화는 확장 제출에 한정되며 다른 이슈의 수동 분류 기준을 바꾸지 않습니다.
2. 메인테이너가 이슈 분류 중에 이슈를 검토하고 별도의 `extension-submission` 레이블을 적용하며, 이것이 자동화된 카탈로그 검증을 시작합니다. 이 공개 저장소에서 기여자는 그 레이블을 직접 적용할 수 없으므로, 레이블을 붙이거나 다시 요청할 것이 없습니다. 이슈는 그저 분류를 기다립니다.
3. 자동화된 워크플로가 제출을 검증하고, 검증을 통과하면 초안 풀 리퀘스트에서 `extensions/catalog.community.json`과 `docs/community/extensions.md`를 갱신합니다
4. 메인테이너가 생성된 풀 리퀘스트를 검토하고 승인되면 병합합니다
5. 확장을 `specify extension search`로 탐색할 수 있게 됩니다

### 메인테이너가 확인하는 것

- 카탈로그 항목 필드가 완전하고 올바른 형식인지
- 다운로드 URL에 접근할 수 있는지
- 저장소가 존재하고 `extension.yml` 매니페스트를 담고 있는지

> [!NOTE]
> 메인테이너는 확장 코드 자체를 검토, 감사, 테스트하지 **않습니다**.

### 기존 확장 업데이트

이미 카탈로그에 있는 확장을 업데이트하려면(예: 새 버전), 갱신된 버전, 다운로드 URL, 그 밖에 바뀐 필드를 담아 새 **[Extension Submission](https://github.com/github/spec-kit/issues/new?template=extension_submission.yml)** 이슈를 제출하세요. 이슈에서 기존 항목에 대한 업데이트임을 언급하세요.

---

<a id="release-workflow"></a>

## 릴리스 워크플로

### 새 버전 게시

새 버전을 릴리스할 때:

1. `extension.yml`의 **버전을 갱신**합니다.

   ```yaml
   extension:
     version: "1.1.0"  # Updated version
   ```

2. **CHANGELOG.md를 갱신**합니다.

   ```markdown
   ## [1.1.0] - 2026-02-15

   ### Added
   - New feature X

   ### Fixed
   - Bug fix Y
   ```

3. **GitHub 릴리스를 만듭니다**.

   ```bash
   git tag v1.1.0
   git push origin v1.1.0
   # Create release on GitHub
   ```

4. 새 버전과 다운로드 URL을 담아 [Extension Submission](https://github.com/github/spec-kit/issues/new?template=extension_submission.yml) 템플릿으로 **업데이트 제출을 등록**합니다. 이슈에서 기존 항목에 대한 업데이트임을 언급하세요.

---

<a id="best-practices"></a>

## 모범 사례

### 확장 설계

1. **단일 책임**: 각 확장은 하나의 도구/연동에 집중해야 합니다
2. **명확한 이름**: 설명적이고 모호하지 않은 이름을 사용하세요
3. **최소한의 의존성**: 불필요한 의존성을 피하세요
4. **하위 호환성**: 시맨틱 버저닝을 엄격히 따르세요

### 문서

1. **README.md 구조**:
   - 개요와 기능
   - 설치 지침
   - 설정 가이드
   - 사용 예시
   - 문제 해결
   - 기여 가이드라인

2. **명령 문서**:
   - 명확한 설명
   - 사전 요구 사항 목록
   - 단계별 지침
   - 오류 처리 안내
   - 예시

3. **설정**:
   - 템플릿 파일 제공
   - 모든 옵션 문서화
   - 예시 포함
   - 기본값 설명

### 보안

1. **입력 검증**: 모든 사용자 입력을 검증하세요
2. **하드코딩된 시크릿 금지**: 자격 증명을 절대 포함하지 마세요
3. **안전한 의존성**: 신뢰할 수 있는 의존성만 사용하세요
4. **정기적인 감사**: 취약점을 확인하세요

### 유지 관리

1. **이슈에 대응**: 메인테이너의 여력이 허락하는 범위에서 보고를 확인하고
   조사하세요
2. **정기적인 업데이트**: 의존성을 최신 상태로 유지하세요
3. **변경 로그**: 상세한 변경 로그를 유지하세요
4. **지원 중단**: 호환성을 깨는 변경은 사전에 알리세요

### 커뮤니티

1. **라이선스**: 허용적인 오픈소스 라이선스(MIT, Apache 2.0)를 사용하세요
2. **기여**: 기여를 환영하세요
3. **행동 강령**: 존중하고 포용하세요
4. **지원**: 도움을 받을 방법(이슈, 토론, 이메일)을 제공하세요

---

## FAQ

### Q: 비공개/독점 확장을 게시할 수 있나요?

A: 메인 카탈로그는 공개 확장 전용입니다. 비공개 확장의 경우:

- 자체 catalog.json 파일을 호스팅하세요
- 릴리스를 검증한 뒤 사용자가 여러분의 카탈로그를 설치 소스로 추가합니다.

  ```bash
  specify extension catalog add https://your-domain.com/catalog.json \
    --name private-catalog --install-allowed
  ```

### Q: 검토에는 얼마나 걸리나요?

A: 제출은 메인테이너의 분류, 자동화된 검증, 생성된 카탈로그 풀 리퀘스트의 검토를
기다립니다. 약속된 처리 기간은 없습니다.

### Q: 제 확장이 거부되면 어떻게 되나요?

A: 무엇을 고쳐야 하는지 피드백을 받게 됩니다. 변경한 뒤 다시 제출하세요.

### Q: 언제든 확장을 업데이트할 수 있나요?

A: 네, 갱신된 버전과 다운로드 URL을 담아 새 [Extension Submission](https://github.com/github/spec-kit/issues/new?template=extension_submission.yml) 이슈를 제출하세요. 기존 항목에 대한 업데이트임을 언급하세요.

### Q: 카탈로그에 오르려면 검증(verified)을 받아야 하나요?

A: 아니요. 모든 커뮤니티 확장은 제출이 검토되고 수락되면 카탈로그에 등재됩니다.

### Q: 확장에 유료 기능을 둘 수 있나요?

A: 확장은 무료이고 오픈소스여야 합니다. 상업적 지원/서비스는 허용되지만 핵심 기능은 무료여야 합니다.

---

## 지원

- **카탈로그 문제**: <https://github.com/github/spec-kit/issues>
- **확장 제출**: [커뮤니티 확장 제출 또는 업데이트](https://github.com/github/spec-kit/issues/new?template=extension_submission.yml)
- **개발 가이드**: [확장 개발하기](developing.md) 참고
- **커뮤니티**: 토론 및 Q&A

---

## 카탈로그 메타데이터

이전 버전을 공개할 때의 [과거 릴리스 기록](catalogs.md#historical-releases)을
포함해 정식 [카탈로그 항목 스키마](catalogs.md#catalog-entry-schema)를 사용하세요.
커뮤니티 제출에는 이슈 양식이 수집하는 모든 설명 필드를 제공하세요.

### 유효한 태그

권장 태그 범주:

- **연동**: jira, linear, github, gitlab, azure-devops
- **범주**: issue-tracking, vcs, ci-cd, documentation, testing
- **플랫폼**: atlassian, microsoft, google
- **기능**: automation, reporting, deployment, monitoring

확장을 가장 잘 설명하는 태그를 2~5개 사용하세요.

---

*최종 업데이트: 2026-01-28*
*카탈로그 형식 버전: 1.0*

## 함께 보기

- [확장 개발하기](developing.md)
- [확장 API 레퍼런스](api.md)
- [확장 카탈로그](catalogs.md)
- [확장 CLI 레퍼런스](cli.md)
- [커뮤니티 확장](../community/extensions.md)
