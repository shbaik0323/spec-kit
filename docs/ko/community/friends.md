# 커뮤니티 프렌즈

> [!NOTE]
> 여기에 나열된 커뮤니티 프로젝트는 각 작성자가 독립적으로 만들고 유지 관리합니다. **퍼스트파티 GitHub 프로젝트**로 명시적으로 표시된 경우가 아니면 GitHub가 **검토하거나, 보증하거나, 지원하지 않습니다**. 설치하기 전에 소스 코드를 검토하고 여러분의 판단에 따라 사용하세요.

Spec Kit을 확장하거나, 시각화하거나, 그 위에 무언가를 만드는 커뮤니티 프로젝트:

- **[Project Room](https://github.com/OpenProjectRoom/desktop)** — 같은 저장소의 로컬 서비스, 테스트, API, MCP 서버, 데이터와 나란히 Spec Kit 계획과 태스크 진행 상황을 보여 주는 오픈소스 Mac 앱입니다. 개발자는 평소 쓰던 에디터에서 계속 코딩하고, Project Room은 실행 중인 앱을 들여다보고 제어할 수 있게 해 줍니다.

- **[cc-spex](https://github.com/rhuss/cc-spex)** — [Superpowers](https://github.com/obra/superpowers) 기반 품질 게이트, 스펙/코드 리뷰, git 워크트리 격리, 에이전트 팀을 통한 병렬 구현과 함께 Spec Kit 위에 합성 가능한 특성(trait)을 추가하는 Claude Code 플러그인입니다.

- **[VS Code Spec Kit Assistant](https://marketplace.visualstudio.com/items?itemName=rfsales.speckit-assistant)** — 단계 상태 시각화, 대화형 태스크 체크리스트, DAG 시각화, 그리고 Claude, Gemini, GitHub Copilot, OpenAI 백엔드 지원과 함께 전체 SDD 워크플로(constitution → specification → planning → tasks → implementation)를 위한 시각적 오케스트레이터를 제공하는 VS Code 확장입니다. PATH에 `specify` CLI가 필요합니다.

- **[SpecKit Assistant](https://www.npmjs.com/package/speckit-assistant)** — 스펙 주도 개발(SDD)을 위한 시각적 오케스트레이터입니다. 로컬 명세, 계획, 태스크 체크리스트를 AI 에이전트(Claude, Gemini, GitHub Copilot)와 연결합니다. 전역 설치가 필요 없으며 `npx speckit-assistant`로 실행하면 됩니다.

- **[SpecKit Companion](https://marketplace.visualstudio.com/items?itemName=alfredoperez.speckit-companion)** — Spec Kit에 시각적 GUI를 제공하는 VS Code 확장입니다. 클릭 가능한 파일 참조가 있는 풍부한 마크다운 뷰어에서 스펙을 둘러보고, 이미지 첨부와 함께 명세를 만들고, 각 단계에 인라인으로 댓글을 달아 다듬고(GitHub 스타일 리뷰), 시각적 단계 표시기로 SDD 워크플로의 진행 상황을 추적하고, 컨스티튜션과 템플릿 같은 방향 설정 문서를 관리하세요.

- **[cc-spec-kit](https://github.com/speckit-community/cc-spec-kit)** — 플러그인 마켓플레이스를 통해 Spec Kit 스킬을 설치하는, 커뮤니티가 유지 관리하는 Claude Code 및 GitHub Copilot CLI용 플러그인입니다.

- **[spectatui](https://github.com/tinesoft/spectatui)** — 기능을 추적하고, 명세·통합·프리셋·워크플로·확장을 관리하고, AI 에이전트 워크플로를 모니터링할 수 있는 Spec Kit용 터미널 UI(TUI) 대시보드입니다. 터미널에서 기존 AI 세션에 연결하거나 새 세션을 시작할 수 있습니다. 키보드와 마우스 지원, 라이트/다크 테마 지원, 사용자 정의 가능하고 성능 지향적입니다. PATH에 `specify` CLI가 필요합니다.

- **[spec-kit-copilot](https://github.com/github/spec-kit-copilot)** — *퍼스트파티 GitHub 프로젝트.* Copilot CLI와 GitHub Copilot 앱 모두에서 Spec Kit `specify` CLI를 Copilot 에이전트에 노출하는 GitHub Copilot **스킬 플러그인**입니다. `specify` 명령 그룹(setup, init, check, extensions, presets, bundles, workflows, workflow steps, self-upgrade)마다 초점을 맞춘 스킬을 제공하므로, 자연어로 Spec Kit 생태계 전체를 탐색하고 구동하며 Copilot이 언제 어떻게 알맞은 `specify` 명령을 대신 실행할지 결정하게 할 수 있습니다.

- **[Spec Kit Workflow Cockpit](https://github.com/markuswondrak/spec-kit-workflow-cockpit)** — Spec Kit 워크플로를 로컬에서 실행하기 위한 터미널 제어판입니다. 파편화된 CLI 흐름을 하나의 실시간 뷰로 대체합니다. 워크플로 전체를 한눈에 보고, 노드별로 진행 상황을 따라가고, 터미널을 떠나지 않고 사람의 리뷰 게이트에서 기능 문서를 읽고 편집하고, 명확하게 제시된 선택지에서 확인과 함께 게이트를 결정할 수 있습니다. 대화형 코딩 세션을 현재 워크플로 실행에 직접 연결하는 `cockpit-run-context` 스킬도 제공합니다.

- **[Specstride](https://specstride.ai/)** — Spec Kit 기능의 `tasks.md`를 입력으로 받고 `spec.md`, `plan.md`, 계약을 읽기 전용 컨텍스트로 사용하는 독립형 구현 오케스트레이터입니다. 코딩 에이전트(Claude Code, Codex, 또는 로컬 모델)가 태스크 단계를 하나씩 진행하게 합니다. 각 단계는 별도의 LLM 비평가가 인수 기준과 코드에 비추어 그 근거를 승인할 때만 넘어갑니다. 막힌 단계는 진단자와 범위를 좁힌 재시도를 받습니다. 선택형 루프가 실행 텔레메트리로부터 단계별 설정을 학습하고, 상황을 악화시키는 변경은 되돌립니다.

- **[Raaghu Spec Kit](https://marketplace.visualstudio.com/items?itemName=Waiin.raaghu-speckit)** — Raaghu Spec Kit은 명세 주도 개발을 Azure DevOps로 가져옵니다. PM, BA, QA가 프로젝트 안의 안내형 AI 워크스페이스에서 구조화된 스펙, 전달 계획, 태스크 분해, 품질 체크리스트를 생성·검토·관리하고, Azure Boards 작업 항목을 만들고 다듬을 수 있습니다. Azure DevOps Services와 Azure DevOps Server(온프레미스)에서 동작합니다.

- **[Entalpa](https://entalpa.com/spec-kit)** — 공개 GitHub 저장소, 또는 zip이나 폴더로 업로드한 비공개 저장소에서 `specs/<feature>/spec.md`와 `tasks.md`를 읽어, 기능들을 하나의 현재 백로그로 합치는 호스팅 웹 앱입니다. 사용자 스토리, 그에 연결되고 `tasks.md`의 상태를 가진 `FR-`/`SC-` 요구사항, 답변된 질문으로서의 명확화 항목이 포함됩니다. 이후의 기능은 사본을 추가하는 대신 자신이 바꾸는 스토리를 수정합니다. 결과를 보는 데는 계정이 필요 없지만, 보관하거나 편집하려면 필요합니다. 업로드는 브라우저에서 읽히며, 그 스펙 파일과 경로, zip이나 폴더 이름만 전송됩니다. 어느 경우든 합치는 과정에서 스펙이 서드파티 AI 제공자에게 전송됩니다. `specs/` 기능으로 다시 내보낼 수 있습니다.
