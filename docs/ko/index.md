<div class="landing-hero">

<img class="landing-hero-logo" src="../images/spec-kit-logo.webp" alt="" width="120" height="120">

# GitHub Spec Kit

**코딩 에이전트와 함께 스펙으로 빌드하고, 버그를 고치고, 아이디어를 평가하세요.**

Spec Kit은 코딩 에이전트에게 의도와 근거가 구현보다 앞서도록 하는 구조화된 프로세스를 제공합니다. 스펙 주도 개발(Spec-Driven Development, SDD), 버그 수정, 아이디어 평가 중 하나로 시작하세요. 프로세스를 단계별로 실행하거나, 자동화하거나, [여러분만의 프로세스를 만들 수도](how-to/customization.md) 있습니다. 소프트웨어 개발을 넘어서는 프로세스도 가능합니다.

<a href="installation.md" class="btn btn-primary btn-lg">Spec Kit 설치하기</a>&nbsp;
<a href="#choose-your-process" class="btn btn-outline-primary btn-lg">프로세스 선택하기</a>

</div>

---

<a id="choose-your-process"></a>

## 프로세스 선택하기

| 필요한 것 | 여기서 시작 | 얻는 것 | 명령 레퍼런스 |
| --- | --- | --- | --- |
| 기능이나 애플리케이션 빌드 | [스펙 주도 개발](quickstart.md) | 계획, 구현, 수렴까지 이어지는 명세 | [Agentic SDD](reference/agentic-sdd.md) |
| 잘못된 동작의 진단과 수리 | [버그 수정](guides/bugfix.md) | 평가된 원인, 범위가 정해진 수정, 기록된 검증 | [Agentic Bug Fix](reference/agentic-bugfix.md) |
| 아이디어에 투자할 가치가 있는지 결정 | [아이디어 평가](guides/assessment.md) | 근거에 기반한 진행, 명확화, 중단 결정 | [Agentic Idea Assessment](reference/agentic-assessment.md) |

이들은 독립적인 진입점이며 필수 단계가 아닙니다. SDD는 코어에 포함되어
제공됩니다. 버그 수정과 평가는 번들로 제공되는 선택형(opt-in) 확장입니다.
평가는 단독으로 쓸 수 있으며, 진행하기로 결정했다고 해서 구현이 자동으로
시작되지는 않습니다.

[Spec Kit 프로젝트가 에이전트 기반 SDLC를 운영하는 방식](guides/agentic-sdlc.md)과
어디에서 Spec Kit 자체를 사용하는지 살펴보세요.

이미 자리 잡은 코드베이스에 Spec Kit을 추가하나요?
[기존 프로젝트 가이드](how-to/existing-projects.md)부터 시작하세요.

<a id="specify-cli"></a>

## Specify CLI

Specify CLI(`specify`)는 프로젝트를 초기화하고 통합, 확장, 프리셋, 워크플로,
워크플로 스텝, 번들을 관리합니다. `/speckit.specify` 같은 프로세스 명령은
터미널이 아니라 코딩 에이전트 안에서 실행됩니다. 에이전트별 구문은
[명령 호출](integrations/cli.md#command-invocation)을, 인자·아티팩트·품질
게이트는 위의 명령 레퍼런스를 참고하세요.

| 공통 기능 | 레퍼런스 |
| --- | --- |
| 프로젝트 초기화, 설치된 도구 확인, 버전 및 시스템 정보 조회 | [핵심 명령](overview/core.md) |
| 실험적 로컬 stdio 서버를 통해 지원되는 Specify 작업을 MCP 클라이언트에 노출 | [MCP 서버](overview/mcp.md) |
| 적용되는 레이어와 가려진 합성 레이어를 포함해 보이는 명령, 템플릿, 스크립트, 훅 조회 | [아티팩트](overview/artifacts.md) |
| 내장, 커뮤니티, 비공개 카탈로그로 컴포넌트를 탐색하고 소스 우선순위와 신뢰 이해 | [카탈로그](overview/catalogs.md) |

---

<div class="pillar-grid">

<div class="pillar-card">

### 기본이 스펙 주도

핵심 SDD 프로세스는 바로 쓸 수 있는 상태로 제공됩니다. **Specify → Plan → Tasks → Implement → Converge**.

만들기 전에 무엇을 만들지 정의하세요. 풍부한 템플릿, 품질 체크리스트, 아티팩트 간 분석이 기본으로 제공됩니다. 각 단계는 다음 단계의 입력이 되는 Markdown 아티팩트를 만들어, AI 코딩 에이전트에게 즉흥적인 프롬프트 대신 구조화된 컨텍스트를 제공합니다.

<a href="quickstart.md" class="pillar-link">워크플로 따라가 보기 →</a>

</div>

<div class="pillar-card">

### 어떤 코딩 에이전트든 사용

<span class="pillar-stat">43개 통합 옵션</span> — 41개의 코딩 에이전트 통합에 더해, 다른 도구를 위한 `generic`과 에이전트 중립 스캐폴딩을 위한 `none`이 있습니다. Copilot, Gemini, Codex, Kilo Code, Zed, Claude, Forge, Kiro 등에서 선택하세요. 명령 하나로 에이전트를 자유롭게 전환할 수 있습니다. 종속(lock-in)이 없습니다.

원하는 에이전트로 `specify init`을 실행하면 Spec Kit이 알맞은 명령 파일과 디렉터리 구조를 자동으로 구성합니다. 사용하는 에이전트가 목록에 없다면 `generic` 통합이 어떤 도구에든 쓸 수 있는 탈출구가 됩니다.

<a href="integrations/index.md" class="pillar-link">모든 통합 보기 →</a>

</div>

<div class="pillar-card">

### 나만의 것으로 만들기

<span class="pillar-stat">186개 커뮤니티 확장</span>, <span class="pillar-stat">42개 커뮤니티 프리셋</span>이 있으며 계속 늘고 있습니다. 프리셋으로 핵심 프로세스를 조정하고, 확장으로 기능을 더하고, 워크플로로 오케스트레이션하고, 이 모두를 공유 가능한 번들로 패키징하세요. 아예 프로세스를 통째로 바꿀 수도 있습니다. 프로세스 자체가 이 구성 요소들 안에 있으므로 SDD에도, 심지어 소프트웨어에도 묶이지 않습니다.

커뮤니티 프로세스, 통합, 품질 게이트를 살펴보세요.

- **[Product Forge](https://github.com/VaiYav/speckit-product-forge)** — 리서치부터 릴리스 준비까지의 제품 라이프사이클 오케스트레이션
- **[Architecture Guard](https://github.com/DyanGalih/architecture-guard)** — 아키텍처 드리프트 감지와 거버넌스
- **[DocGuard](https://github.com/raccioly/docguard)** — 문서 무결성, 추적성, 라이프사이클 점검
- **[Quality Gates](https://github.com/schwichtgit/spec-gates)** — 에이전트, Git, CI 경계에서의 정책 기반 점검
- **[Epic](https://github.com/pmckeown/spec-kit-epic)** — 큰 이니셔티브를 순서가 있고 조율된 기능들로 분할
- **[Linear Weave](https://github.com/tonydwoodhouse/spec-kit-linear-weave)** — 요구사항, 태스크, 상태를 Linear와 연결

소프트웨어 밖에서는 **[Fiction Book Writing 프리셋](https://github.com/adaumann/speckit-preset-fiction-book-writing)**이 스토리 바이블부터 투고까지 소설과 장편 픽션 집필을 지원합니다.

독립적으로 유지 관리되는 이 커뮤니티 프로젝트들을 소개하는 것은 감사(audit)나 보증이 아닙니다. 설치하고 사용하기 전에 소스 코드를 검토하세요.

<a href="presets/overview.md" class="pillar-link">프리셋 →</a>&nbsp;&nbsp;
<a href="extensions/overview.md" class="pillar-link">확장 →</a>&nbsp;&nbsp;
<a href="workflows/overview.md" class="pillar-link">워크플로 →</a>&nbsp;&nbsp;
<a href="workflow-steps/overview.md" class="pillar-link">워크플로 스텝 →</a>&nbsp;&nbsp;
<a href="bundles/overview.md" class="pillar-link">번들 →</a>

</div>

<div class="pillar-card">

### 조직에 통합하기

오프라인, 방화벽 내부, 그리고 **Windows, macOS, Linux**에서 동작합니다. 자체 카탈로그를 호스팅해 조직이 탐색하고 권장하는 통합, 확장, 프리셋, 워크플로, 번들을 선별하세요.

Quality Gates와 Architecture Guard 같은 커뮤니티 확장은 팀이 이미 일하는 방식에 맞는 정책 점검과 거버넌스를 더해 줍니다.

<a href="install/air-gapped.md" class="pillar-link">엔터프라이즈 / 에어갭 →</a>&nbsp;&nbsp;
<a href="overview/catalogs.md" class="pillar-link">카탈로그 →</a>

</div>

</div>

---

<div class="community-section">

## 커뮤니티가 만듭니다

**320명 이상의 기여자**가 Spec Kit을 함께 만들었습니다. 더 넓은 커뮤니티는 통합부터 완전히 새로운 프로세스까지 무엇이든 만듭니다. 누구나 확장, 프리셋, 워크플로를 만들어 게시할 수 있습니다.

<div class="stats-grid">
  <div class="stat-item">
    <span class="stat-number">147K+</span>
    <span class="stat-label">GitHub 스타</span>
  </div>
  <div class="stat-item">
    <span class="stat-number">320+</span>
    <span class="stat-label">기여자</span>
  </div>
  <div class="stat-item">
    <span class="stat-number">43</span>
    <span class="stat-label">통합 옵션</span>
  </div>
  <div class="stat-item">
    <span class="stat-number">186</span>
    <span class="stat-label">커뮤니티 확장</span>
  </div>
  <div class="stat-item">
    <span class="stat-number">42</span>
    <span class="stat-label">커뮤니티 프리셋</span>
  </div>
  <div class="stat-item">
    <span class="stat-number">11</span>
    <span class="stat-label">프렌드 프로젝트</span>
  </div>
</div>

<a href="community/presets.md">프리셋</a> · <a href="community/walkthroughs.md">워크스루</a> · <a href="community/friends.md">프렌즈</a>

</div>

---

## 문서 둘러보기

<div class="nav-cards">
  <a href="guides/agentic-sdlc.md" class="nav-card">
    <strong>Spec Kit의 에이전트 기반 SDLC</strong>
    <span>이 프로젝트가 변경을 평가, 빌드, 테스트, 출시, 수리하는 방법</span>
  </a>
  <a href="quickstart.md" class="nav-card">
    <strong>스펙 주도 개발</strong>
    <span>기능을 정의, 계획, 구현하고 수렴시키기</span>
  </a>
  <a href="guides/bugfix.md" class="nav-card">
    <strong>버그 수정</strong>
    <span>원인을 평가하고, 범위가 정해진 수정을 적용하고, 결과를 검증하기</span>
  </a>
  <a href="guides/assessment.md" class="nav-card">
    <strong>아이디어 평가</strong>
    <span>아이디어를 조사하고 투자할 가치가 있는지 결정하기</span>
  </a>
  <a href="how-to/customization.md" class="nav-card">
    <strong>Spec Kit을 여러분의 프로세스에 맞추기</strong>
    <span>프로세스를 조정하고, 기능을 더하고, 역할 기반 구성을 패키징하기</span>
  </a>
  <a href="how-to/existing-projects.md" class="nav-card">
    <strong>기존 프로젝트</strong>
    <span>이미 자리 잡은 코드베이스에 Spec Kit을 안전하게 도입하기</span>
  </a>
  <a href="upgrade.md" class="nav-card">
    <strong>업그레이드</strong>
    <span>기존 Spec Kit 프로젝트를 릴리스에 맞춰 최신 상태로 유지하기</span>
  </a>
  <a href="overview/core.md" class="nav-card">
    <strong>핵심 명령</strong>
    <span>프로젝트 초기화, 도구 확인, 버전 및 시스템 정보 조회</span>
  </a>
  <a href="community/overview.md" class="nav-card">
    <strong>커뮤니티</strong>
    <span>커뮤니티 통합, 확장, 프리셋, 워크플로, 스텝, 번들과 관련 리소스</span>
  </a>
  <a href="how-to/local-development.md" class="nav-card">
    <strong>Spec Kit 로컬 개발</strong>
    <span>Spec Kit 자체에 대한 변경을 실행, 디버그, 점검하기</span>
  </a>
  <a href="how-to/index.md" class="nav-card">
    <strong>How to</strong>
    <span>Spec Kit 실무를 이해하고 프로젝트 작업에 적용하기</span>
  </a>
  <a href="overview/history.md" class="nav-card">
    <strong>역사</strong>
    <span>Spec Kit이 SDD 기반에서 확장 가능한 프로세스 하네스로 성장한 과정</span>
  </a>
</div>

---

<div class="footer-cta">

```bash
uv tool install specify-cli
specify init my-project --integration copilot
```

시작할 준비가 되셨나요? [프로세스를 선택하세요](#choose-your-process).

</div>

## 함께 보기

각 컴포넌트에는 자체 가이드와 관리 레퍼런스가 있습니다.

| 컴포넌트 | 가이드 | CLI 레퍼런스 |
| --- | --- | --- |
| 통합 | [에이전트 설정과 전달](integrations/index.md) | [통합 관리](integrations/cli.md) |
| 확장 | [기능, 명령, 훅 추가](extensions/overview.md) | [확장 관리](extensions/cli.md) |
| 프리셋 | [템플릿, 명령, 스크립트 사용자 정의](presets/overview.md) | [프리셋 관리](presets/cli.md) |
| 워크플로 | [반복 가능하고 재개 가능한 프로세스 자동화](workflows/overview.md) | [워크플로 관리](workflows/cli.md) |
| 워크플로 스텝 | [내장 동작 재사용 또는 사용자 정의 타입 개발](workflow-steps/overview.md) | [스텝 관리](workflow-steps/cli.md) |
| 번들 | [기존 컴포넌트로 이루어진 버전 지정 구성 설치](bundles/overview.md) | [번들 관리](bundles/cli.md) |

비공개 카탈로그, 다운로드, 릴리스 확인에 대해서는
[인증 설정](reference/authentication.md)을 참고하세요. 어떤 구성 요소가
필요에 맞는지 고르려면
[Spec Kit을 여러분의 프로세스에 맞추기](how-to/customization.md)를 참고하세요.

<p class="text-end small text-body-secondary">최종 업데이트: 2026년 11월 4일</p>
