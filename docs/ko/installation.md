# 설치 가이드

## 사전 요구 사항

- **Linux/macOS** (또는 Windows. 이제 WSL 없이도 PowerShell 스크립트를 지원합니다)
- [지원되는 AI 코딩 에이전트](integrations/index.md#choose-an-integration).
  설치 요구 사항은 사용하는 통합의 페이지를 참고하거나, 에이전트 중립
  스캐폴딩을 위해 [`none`](integrations/none.md)을 사용하세요.
- 패키지 관리를 위한 [uv](https://docs.astral.sh/uv/)(권장) 또는 영구 설치를 위한 [pipx](https://pipx.pypa.io/)
- [Python 3.11+](https://www.python.org/downloads/)
- [Git](https://git-scm.com/downloads) *(선택 사항 — git 확장을 활성화한 경우에만 필요)*

## 설치

> [!IMPORTANT]
> Spec Kit은 두 개의 공식 채널로 배포되며, 둘 다 Spec Kit 메인테이너가 게시하고 유지 관리합니다. [github/spec-kit](https://github.com/github/spec-kit) GitHub 저장소(소스 설치)와 [PyPI](https://pypi.org/project/specify-cli/)의 [`specify-cli`](https://pypi.org/project/specify-cli/) 패키지입니다. 일반적인 설치에는 두 경로 모두 지원되므로 아래에 나온 명령을 사용하세요. 설치 후에는 로컬 버전/런타임 점검 삼아 `specify version`을 실행하세요. 이 명령은 `specify` 명령을 사용할 수 있는지 확인하고 버전을 알려 주지만, 실행 파일이 PyPI에서 왔는지 GitHub에서 왔는지를 증명하지는 않습니다. 오프라인 또는 에어갭 환경에서는 이 저장소에서 로컬로 빌드한 휠도 유효합니다.

Spec Kit은 두 가지 설치 경로를 지원합니다.

1. **소스에서 설치(GitHub)** — 릴리스 태그에 고정하는 권장 경로입니다.
2. **PyPI에서 설치** — 평소 쓰는 Python 도구로 게시된 `specify-cli` 패키지를 설치합니다.

<a id="install-from-source--persistent-installation-recommended"></a>

### 소스에서 설치 — 영구 설치(권장)

한 번 설치해 어디서나 사용하세요. `vX.Y.Z`를 [Releases](https://github.com/github/spec-kit/releases)의 릴리스 태그로 바꾸되, 앞의 `v`는 유지하세요(예: `0.12.11`이 아니라 `v0.12.11`).

> [!NOTE]
> 아래 명령에는 **[uv](https://docs.astral.sh/uv/)**가 필요합니다. `command not found: uv`가 표시되면 [먼저 uv를 설치하세요](./install/uv.md).

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z
```

그런 다음 프로젝트를 초기화합니다.

```bash
specify init <PROJECT_NAME> --integration copilot
```

### PyPI에서 설치

Spec Kit은 PyPI에도 [`specify-cli`](https://pypi.org/project/specify-cli/)로 게시되므로, Git URL을 참조하지 않고 선호하는 Python 패키지 관리자로 설치할 수 있습니다.

```bash
# uv 사용(권장)
uv tool install specify-cli

# 또는 pipx 사용
pipx install specify-cli

# 또는 pip 사용
pip install specify-cli
```

`specify-cli`가 이미 설치되어 있으면, 더 새로운 릴리스가 있더라도 이 명령들은 설치된 버전을 그대로 유지합니다. `specify version`이 보여 주는 버전을 [PyPI의 최신 버전](https://pypi.org/project/specify-cli/)과 비교하세요. 오래된 설치를 교체하려면 원래 설치에 사용한 것과 같은 도구를 사용하세요. `uv tool install --force specify-cli`, `pipx install --force specify-cli`, 또는 `pip install --upgrade specify-cli`입니다. 다른 도구를 쓰면 두 번째 사본이 설치되고 예전 `specify`가 `PATH`에서 여전히 먼저 잡힐 수 있습니다. 오래된 CLI는 현재 릴리스로 프로젝트를 구성할 수 없습니다. 예를 들어 0.1.6에서는 `specify init --integration ...`이 `No such option: --integration`으로 실패하고, `specify init --ai ...`는 `No matching release asset found`로 실패합니다.

특정 릴리스를 설치하려면 버전을 고정하세요. 예: `uv tool install specify-cli==0.12.11`. 업그레이드 방법과 [사용자 정의 또는 비공개 패키지 인덱스에서 설치하는 방법](install/pypi.md#install-from-a-custom-or-private-package-index)을 포함한 자세한 내용은 [PyPI 설치 가이드](install/pypi.md)를 참고하세요.

### 일회성 사용

설치하지 않고 바로 실행하려면 [일회성 사용(uvx)](install/one-time.md) 가이드를 참고하세요.

### 다른 패키지 관리자

- **PyPI** — [PyPI 설치 가이드](install/pypi.md) 참고
- **pipx** — [pipx 설치 가이드](install/pipx.md) 참고
- **엔터프라이즈 / 에어갭** — [에어갭 설치 가이드](install/air-gapped.md) 참고

### 통합 지정하기

대화형 터미널에서는 초기화 중에 코딩 에이전트 통합을 선택하라는 프롬프트가 표시됩니다. CI나 파이프 실행 같은 비대화형 세션에서는 `--integration`을 전달하지 않으면 GitHub Copilot이 기본값입니다.

초기화할 때 코딩 에이전트 통합을 미리 지정할 수 있습니다.

```bash
specify init <project_name> --integration claude
specify init <project_name> --integration gemini
specify init <project_name> --integration copilot
specify init <project_name> --integration codebuddy
specify init <project_name> --integration pi
specify init <project_name> --integration omp
```

README의 예시는 GitHub Copilot의 기본 스킬 모드를 사용합니다. 지원되는 다른
에이전트를 사용하려면 `copilot`을 해당
[통합 키](integrations/index.md)로 바꾸세요. 각 에이전트와 모드에서 사용하는
구문은 [명령 호출](integrations/cli.md#command-invocation)을 참고하세요.

### 스크립트 유형 지정하기(Shell, PowerShell 또는 Python)

자동화 스크립트는 Bash(`.sh`), PowerShell(`.ps1`), Python(`.py`) 변형으로 제공됩니다.

자동 동작:

- Windows 기본값: `ps`
- 그 외 OS 기본값: `sh`
- 대화형 모드: `--script`를 전달하지 않으면 프롬프트가 표시됩니다

특정 스크립트 유형을 강제하려면 다음과 같이 합니다.

```bash
specify init <project_name> --script sh
specify init <project_name> --script ps
specify init <project_name> --script py
```

### CI 및 AI 에이전트 하네스

키보드가 없거나 PTY가 화살표 키를 보낼 수 없는 환경에서는
`--non-interactive`를 전달해 초기화가 선택기에서 대기하지 않도록 하세요.

```bash
specify init my-project --non-interactive --ignore-agent-tools
```

`--ignore-agent-tools`는 에이전트가 다른 곳에서 실행될 때 에이전트 실행 파일
확인을 건너뜁니다. 확인이 필요하면 생략하세요. 비대화형 초기화는 다른 통합을
명시적으로 선택하지 않는 한 Copilot을 기본값으로 사용합니다.

비어 있지 않은 디렉터리에서는 `--force`가 병합 경고를 승인합니다. 초기화하기
전에 [기존 프로젝트 가이드](how-to/existing-projects.md)를 검토하고 로컬
변경 사항을 커밋하거나 백업하세요.

```bash
specify init --here --force --non-interactive --integration claude
```

### 에이전트 도구 확인 건너뛰기

적절한 도구가 있는지 확인하지 않고 템플릿만 받고 싶다면 다음과 같이 합니다.

```bash
specify init <project_name> --integration claude --ignore-agent-tools
```

## 확인

설치 후 로컬 버전/런타임 점검으로 다음 명령을 실행하세요.

```bash
specify version
```

이 명령은 `specify` 명령을 사용할 수 있고 예상한 버전을 보고하는지 확인합니다. 해당 실행 파일이 PyPI에서 왔는지 GitHub에서 왔는지를 증명하지는 않습니다.

**최신 상태 유지:** 주기적으로 `specify self check`를 실행해 더 새로운 릴리스가 있는지 확인하세요. 이 명령은 읽기 전용이며 설치를 절대 수정하지 않습니다. 업그레이드할 준비가 되면 [업그레이드 가이드](./upgrade.md)를 따르세요.

GitHub Copilot의 기본 스킬 모드로 초기화하면 다음 스킬을 사용할 수 있어야
합니다. 터미널이 아니라 에이전트의 채팅에서 호출하세요.

- `/speckit-specify` - 명세 작성
- `/speckit-plan` - 구현 계획 생성
- `/speckit-tasks` - 실행 가능한 태스크로 분해
- `/speckit-implement` - 구현 태스크 실행
- `/speckit-analyze` - 아티팩트 간 일관성 검증
- `/speckit-clarify` - 모호한 부분 식별 및 해소
- `/speckit-checklist` - 품질 체크리스트 생성
- `/speckit-constitution` - 프로젝트 원칙 생성 또는 갱신
- `/speckit-converge` - 아티팩트에 비추어 코드베이스를 평가하고 남은 태스크 추가

스크립트는 선택한 스크립트 유형에 해당하는 변형 하위 디렉터리에 설치됩니다.

- `.specify/scripts/bash/` — `.sh` 스크립트 포함(Linux/macOS 기본값)
- `.specify/scripts/powershell/` — `.ps1` 스크립트 포함(Windows 기본값)
- `.specify/scripts/python/` — `.py` 스크립트 포함(`--script py`로 선택. 플랫폼 셸 폴백도 함께 설치됨)

## 프로세스 선택하기

초기화가 끝나면 프로젝트 디렉터리에서 코딩 에이전트를 실행하고
[SDD 빠른 시작](quickstart.md),
[버그 수정 빠른 시작](guides/bugfix.md), 또는
[아이디어 평가 빠른 시작](guides/assessment.md)을 따르세요. 이들은 독립적인
진입점입니다. SDD는 바로 사용할 수 있고, 나머지 가이드는 번들로 제공되는
선택형 확장을 설치하는 방법을 보여 줍니다.

## 문제 해결

### 엔터프라이즈 / 에어갭 설치

환경에서 PyPI나 GitHub 접근이 차단되어 있다면, 휴대 가능한 휠 번들을 만드는 단계별 지침이 담긴 [엔터프라이즈 / 에어갭 설치](install/air-gapped.md) 가이드를 참고하세요.

### Linux의 Git Credential Manager

Linux에서 Git 인증에 문제가 있다면 [에어갭 설치 가이드](install/air-gapped.md#git-credential-manager-on-linux)에서 Git Credential Manager 설정 지침을 참고하세요.

> 이 페이지의 에이전트 호출 예시는 GitHub Copilot의 기본 스킬 구문(`/speckit-*`)을 사용합니다. 다른 통합과 모드는 [명령 호출](integrations/cli.md#command-invocation)을 참고하세요.
