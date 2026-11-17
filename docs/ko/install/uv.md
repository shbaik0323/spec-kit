# uv 설치하기

[uv](https://docs.astral.sh/uv/)는 [Astral](https://astral.sh/)이 만든 빠른 Python 패키지 관리자입니다. Spec Kit은 전역 Python 환경을 어지럽히지 않고 `specify` CLI를 실행하기 위해 `uv`(`uvx` 또는 `uv tool install`을 통해)를 사용합니다.

> [!NOTE]
> **이미 uv가 있나요?** `uv --version`을 실행해 설치되어 있는지 확인한 다음 [설치 가이드](../installation.md)로 돌아가세요.

## 설치

### macOS 및 Linux — 독립 실행형 설치 프로그램

macOS나 Linux에 uv를 설치하는 가장 빠른 방법은 공식 셸 스크립트입니다.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

스크립트가 끝나면 설치 프로그램이 출력하는 지침에 따라 uv를 `PATH`에 추가한 다음 새 터미널을 여세요.

### Windows — 독립 실행형 설치 프로그램

**명령 프롬프트 또는 PowerShell**에서 다음을 실행합니다.

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

스크립트가 끝나면 `uv` 바이너리가 `PATH`에 잡히도록 새 터미널을 여세요.

### macOS — Homebrew

```bash
brew install uv
```

### Windows — WinGet

```powershell
winget install --id=astral-sh.uv -e
```

### Windows — Scoop

```powershell
scoop install uv
```

## 확인

uv가 설치되어 있고 `PATH`에 있는지 확인합니다.

```bash
uv --version
```

`uv 0.x.y (...)`와 비슷한 출력이 표시되어야 합니다.

## 더 읽을거리

고급 옵션(자체 업데이트, 프록시 설정, 제거 등)은 공식 [uv 설치 문서](https://docs.astral.sh/uv/getting-started/installation/)를 참고하세요.
