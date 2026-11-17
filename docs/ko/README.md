# 문서

이 폴더에는 [DocFX](https://dotnet.github.io/docfx/)로 빌드하는 Spec Kit 문서의 소스 파일이 들어 있습니다.

## 로컬에서 빌드하기

문서를 로컬에서 빌드하려면 다음과 같이 합니다.

1. DocFX를 설치합니다.

   ```bash
   dotnet tool install -g docfx
   ```

2. 문서를 빌드합니다.

   ```bash
   cd docs
   docfx docfx.json --serve
   ```

3. 브라우저에서 `http://localhost:8080`을 열어 문서를 확인합니다.

## 구조

목차는 프로젝트 배경과 공통 CLI 레퍼런스를 **Overview** 아래에 묶습니다.
**Getting Started**는 각 프로세스 안내서를 해당 명령 레퍼런스와 짝지어 둡니다.
**How to**는 프로젝트 실무, 인증, 그리고 Spec Kit 자체의 로컬 개발을 다룹니다.
통합, 확장, 프리셋, 워크플로, 워크플로 스텝, 번들, 커뮤니티 리소스는 각자의
섹션을 유지합니다.

Overview의 보조 페이지는 `overview/` 아래에 있으며, `index.md`는 사이트
홈페이지로 남아 있습니다. 프로세스 명령 레퍼런스와 인증 문서는 `reference/`
아래에, 로컬 개발 가이드는 `how-to/` 아래에 있습니다.

- `docfx.json` - DocFX 설정 파일
- `index.md` - 문서 메인 홈페이지
- `overview/history.md` - 프로젝트 역사와 관리 주체
- `toc.yml` - 목차 설정
- `overview/core.md` - 핵심 CLI 명령과 공통 프로젝트 설정
- `overview/mcp.md` - 실험적 로컬 stdio MCP 서버
- `overview/artifacts.md` - 아티팩트 목록과 합성 레퍼런스
- `reference/authentication.md` - 인증 설정 가이드 및 레퍼런스
- `how-to/local-development.md` - Spec Kit을 로컬에서 개발하고 점검하기
- `installation.md` - 설치 가이드
- `quickstart.md` - 스펙 주도 개발 안내서
- `guides/agentic-sdlc.md` - Spec Kit이 에이전트 기반 및 전통적 SDLC 실무를 스스로에게 적용하는 방법
- `guides/bugfix.md` - 버그 수정 안내서
- `guides/assessment.md` - 아이디어 평가 안내서
- `how-to/index.md` - How-to 가이드 목록
- `how-to/sdd.md` - 스펙 주도 개발 이해하기
- `how-to/existing-projects.md` - 기존 프로젝트에 Spec Kit 도입하기
- `how-to/monorepo.md` - 모노레포에서 작업하기
- `how-to/contract-driven-development.md` - 컴포넌트 계약을 정의하고 발전시키기
- `how-to/complex-features.md` - 복잡한 기능을 범위가 정해진 실행 단위로 구현하기
- `how-to/spec-of-specs.md` - 큰 기능을 더 작은 명세로 나누기
- `how-to/persisting-specs.md` - 전달 이후 명세 보존하기
- `how-to/evolving-specs.md` - 후속 변경을 위해 명세 발전시키기
- `how-to/customization.md` - Spec Kit을 여러분의 프로세스에 맞추기
- `overview/catalogs.md` - 카탈로그 탐색, 신뢰, 비공개 소스, 우선순위
- `extensions/overview.md` - 확장 사용자 가이드
- `extensions/cli.md` - 확장 관리 CLI 레퍼런스
- `extensions/catalogs.md` - 확장 카탈로그 설정, 신뢰, 릴리스 스키마
- `extensions/developing.md` - 확장 작성 가이드
- `extensions/api.md` - 확장 매니페스트, API, 훅, 런타임 이벤트 계약
- `extensions/publishing.md` - 확장 게시 및 카탈로그 제출 가이드
- `presets/overview.md` - 프리셋 사용자 가이드
- `presets/cli.md` - 프리셋 관리 CLI 레퍼런스
- `presets/developing.md` - 프리셋 작성 및 로컬 테스트 가이드
- `presets/api.md` - 매니페스트, 합성, 라이프사이클 레퍼런스
- `presets/catalogs.md` - 프리셋 카탈로그 설정, 스키마, 신뢰
- `presets/publishing.md` - 프리셋 게시 및 카탈로그 제출 가이드
- `workflows/overview.md` - 워크플로 사용자 가이드와 실행/검토/재개 안내
- `workflows/cli.md` - 워크플로, 오버레이, 워크플로 카탈로그 CLI 레퍼런스
- `workflows/developing.md` - 워크플로 작성 가이드
- `workflows/api.md` - YAML, 실행, 영속 상태 계약
- `workflows/expressions.md` - 표현식 언어, 런타임 컨텍스트, 셸 안전성
- `workflows/overlays.md` - 오버레이와 슬롯을 통한 워크플로 합성
- `workflows/catalogs.md` - 워크플로 카탈로그, 릴리스, 신뢰
- `workflows/publishing.md` - 워크플로 게시 및 카탈로그 제출 가이드
- `workflow-steps/overview.md` - 내장 및 사용자 정의 워크플로 스텝 개요
- `workflow-steps/cli.md` - 사용자 정의 스텝 및 스텝 카탈로그 관리 CLI
- `workflow-steps/developing.md` - 스텝 패키지 작성과 검증
- `workflow-steps/api.md` - Python 확장 API, 레지스트리, 로더 계약
- `workflow-steps/types.md` - 내장 스텝 설정과 출력
- `workflow-steps/catalogs.md` - 스텝 카탈로그, 릴리스, 신뢰
- `workflow-steps/publishing.md` - 스텝 릴리스 및 제출 안내
- `bundles/overview.md` - 번들 사용자 가이드와 퍼스트파티 구성
- `bundles/cli.md` - 번들 관리 CLI 레퍼런스
- `bundles/developing.md` - 번들 작성 및 설치 테스트 가이드
- `bundles/api.md` - 매니페스트, 참조 확장, 설치 기록 계약
- `bundles/catalogs.md` - 번들 카탈로그, 정책, 릴리스 이력
- `bundles/publishing.md` - 번들 게시 및 카탈로그 제출 가이드
- `community/extensions.md` - 커뮤니티 확장 목록
- `community/integrations.md` - 커뮤니티 통합 탐색과 기여
- `community/workflows.md` - 커뮤니티 워크플로 목록과 기여
- `community/workflow-steps.md` - 커뮤니티 스텝 목록과 제출 계약
- `integrations/index.md` - 통합 목록과 공통 설정 안내
- `integrations/cli.md` - 공통 통합 관리 CLI 레퍼런스
- `integrations/api.md` - 공개 Python 어댑터와 파일 소유권 계약
- `integrations/<key>.md` - 에이전트별 설정, 호출, 옵션, 제한 사항
- `integrations/catalog.md` - 카탈로그 스키마와 외부 어댑터 메타데이터
- `integrations/contributing.md` - 통합 및 카탈로그 기여 가이드
- `reference/agentic-*.md` - 상세 프로세스 명령 레퍼런스
- `_site/` - 생성된 문서 출력물(git에서 무시됨)
- `ko/` - 이 구조를 그대로 따르는 한국어 번역. 자체 사이드바 목차는 `ko/toc.yml`

## 배포

`main` 브랜치에 변경이 푸시되면 문서가 자동으로 빌드되어 GitHub Pages에 배포됩니다. 워크플로는 `.github/workflows/docs.yml`에 정의되어 있습니다.
