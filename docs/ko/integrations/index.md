# 통합

통합은 명령, 스킬, 레시피를 통해 Spec Kit을 AI 코딩 에이전트에 연결합니다.
에이전트 없이 공유 스캐폴딩만 원한다면 [none](none.md)을 사용하세요.
사전 요구 사항, 설정, 호출 구문, 통합별 옵션이나 제한 사항은 아래에서 통합을
선택해 확인하세요.

## 에이전트 설정하기

CLI 사전 요구 사항과 설치는 [설치 가이드](../installation.md)를, 첫 프로세스는
[SDD 빠른 시작](../quickstart.md)을 참고하세요.

새 프로젝트의 경우:

```bash
specify init my-project --integration <key>
```

프로세스 명령은 터미널이 아니라 코딩 에이전트 안에서 호출하세요.
문서에서는 보통 `/speckit.*` 표기를 사용하지만, 에이전트에 따라 하이픈으로 연결된
스킬, `$` 스킬, 또는 다른 네이티브 형식을 사용할 수 있습니다. 사용하는 통합의
페이지와 [명령 호출 비교](cli.md#command-invocation)를 참고하세요.

## 다른 에이전트 추가하기

초기화된 프로젝트에서는
[`specify integration install <key>`](cli.md#install-an-integration)를 사용하세요.
에이전트 가이드에서 다중 설치 안전성 상태를 확인하고
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을
따르세요.

다른 통합을 설치해도 기본값은 바뀌지 않습니다. 활성화하려면
[`specify integration use <key>`](cli.md#use-an-installed-integration)를
사용하세요.

## 에이전트 제거하기

[`specify integration uninstall <key>`](cli.md#uninstall-an-integration)를 사용해
에이전트 애플리케이션 자체가 아니라 에이전트의 Spec Kit 통합을 제거하세요.
수정된 파일 처리와 정리 예외는 제거 레퍼런스를 참고하세요.

통합을 교체하려면 대신
[`specify integration switch <key>`](cli.md#switch-to-a-different-integration)를
사용하세요.

<a id="working-in-a-team"></a>

## 팀에서 작업하기

여러 에이전트를 쓰는 팀은 에이전트 중립 프로젝트에서 시작할 수 있습니다.

```bash
specify init my-project --integration none
```

각 개발자는 로컬 체크아웃에서 자신이 고른 에이전트로 전환합니다.

```bash
specify integration switch <key>
```

개발이 끝나면 커밋을 스테이징하기 전에 다시 전환합니다.

```bash
specify integration switch none
```

> [!IMPORTANT]
> Spec Kit이 생성한 에이전트 명령이나 스킬을 수동으로 편집하지 마세요. 사용자
> 정의는 검토를 거친 [프리셋](../presets/overview.md)이나
> [프로젝트 로컬 오버라이드](../how-to/customization.md#project-local-overrides-and-resolution)로
> 유지 관리하세요. 커밋하기 전에
> [`specify integration status`](cli.md#report-integration-status)를 사용해
> `none`이 유일하게 설치된 기본 통합인지 확인하세요. Git diff와 남아 있는 에이전트
> 파일을 검토하세요.

팀은 커밋된 기준선에 대한 CI 검사와 스테이징된 파일에 대한 선택적 pre-commit
검사를 추가할 수 있습니다.

<a id="choose-an-integration"></a>

## 통합 선택하기

아래에 표시된 키를 `--integration <key>`에 정확히 그대로 사용하세요. CLI 기반
통합은 에이전트의 CLI가 설치되어 있어야 하고, IDE 기반 통합은 IDE의 에이전트 채팅
창을 통해 실행됩니다. 각 페이지는 에이전트의 설치 문서로 연결됩니다.

| 통합 | 키 |
| --- | --- |
| [Alquimia AI](alquimia.md) | `alquimia` |
| [Amp](amp.md) | `amp` |
| [Antigravity](agy.md) | `agy` |
| [Auggie CLI](auggie.md) | `auggie` |
| [Claude Code](claude.md) | `claude` |
| [Cline](cline.md) | `cline` |
| [CodeBuddy CLI](codebuddy.md) | `codebuddy` |
| [Codex CLI](codex.md) | `codex` |
| [Command Code](command-code.md) | `command-code` |
| [Cursor](cursor-agent.md) | `cursor-agent` |
| [DeepSeek Harness](dsh.md) | `dsh` |
| [Devin for Terminal](devin.md) | `devin` |
| [Docker Agent](docker-agent.md) | `docker-agent` |
| [Factory Droid](droid.md) | `droid` |
| [Firebender](firebender.md) | `firebender` |
| [Forge](forge.md) | `forge` |
| [Gemini CLI](gemini.md) | `gemini` |
| [GitHub Copilot](copilot.md) | `copilot` |
| [Goose](goose.md) | `goose` |
| [Grok Build](grok.md) | `grok` |
| [Hermes](hermes.md) | `hermes` |
| [IBM Bob](bob.md) | `bob` |
| [Junie](junie.md) | `junie` |
| [Kilo Code](kilocode.md) | `kilocode` |
| [Kimi Code](kimi.md) | `kimi` |
| [Kiro CLI](kiro-cli.md) | `kiro-cli` |
| [Lingma](lingma.md) | `lingma` |
| [MiniMax Code](mcode.md) | `mcode` |
| [Mistral Vibe](vibe.md) | `vibe` |
| [Muse Code](muse.md) | `muse` |
| [None: 공유 스캐폴딩만](none.md) | `none` |
| [Oh My Pi](omp.md) | `omp` |
| [opencode](opencode.md) | `opencode` |
| [Pi Coding Agent](pi.md) | `pi` |
| [Qoder CLI](qodercli.md) | `qodercli` |
| [Qwen Code](qwen.md) | `qwen` |
| [RovoDev](rovodev.md) | `rovodev` |
| [SHAI (OVHcloud)](shai.md) | `shai` |
| [Tabnine CLI](tabnine.md) | `tabnine` |
| [Trae](trae.md) | `trae` |
| [ZCode](zcode.md) | `zcode` |
| [Zed](zed.md) | `zed` |
| [Generic: 직접 가져온 에이전트 사용](generic.md) | `generic` |

## 공통 관리와 외부 통합

[통합 CLI 레퍼런스](cli.md)는 공통 설치, 업그레이드, 제거, 상태, 카탈로그, 기본값
선택 동작을 문서화합니다. 이 규칙들은 각 페이지에서 반복하지 않고 모든 통합에
공통으로 적용됩니다. 개별 가이드는 워크플로 CLI 디스패치가 지원되는지도 밝힙니다.
CLI가 필요 없는 IDE 설정이라고 해서 워크플로 스텝이 CLI 없이 실행될 수 있다는
뜻은 아닙니다.

여기에 나열되지 않은 에이전트에는 [generic 통합](generic.md)을 사용하거나
[외부 어댑터를 탐색](cli.md#search-available-integrations)하세요.
외부 어댑터 코드는 실행을 승인하기 전에 검토하세요. 카탈로그 탐색은 보안 보증이
아니며, 기본 커뮤니티 카탈로그는 탐색 전용입니다. 신뢰 및 설치 요구 사항은
[카탈로그로 설치하는 외부 어댑터](cli.md#catalog-installed-external-adapters)를
참고하세요.

[API 레퍼런스](api.md)는 지원되는 Python 어댑터 계약을 문서화합니다.
[카탈로그 레퍼런스](catalog.md)는 디스크립터와 카탈로그 메타데이터를 설명합니다.
내장 통합을 개발하거나 외부 어댑터를 게시하려면
[통합 기여하기](contributing.md)를 참고하세요.

## 함께 보기

- [커뮤니티 통합](../community/integrations.md)
