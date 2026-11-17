# Spec Kit을 로컬에서 개발하기

체크아웃이나 워크트리마다 별도의 `.venv`를 사용하고, 그 체크아웃을 편집 가능
모드로 설치하고, 명시적인 환경 경로를 통해 CLI와 테스트를 실행하세요. 이렇게 하면
로컬 변경이 전역 설치된 릴리스 및 다른 Spec Kit 체크아웃으로부터 격리됩니다. 환경
활성화는 필요하지 않습니다.

## 1. 체크아웃 준비

[Python 3.11+](https://www.python.org/downloads/),
[uv](https://docs.astral.sh/uv/), [Git](https://git-scm.com/downloads/)을
설치하세요. 포크를 클론하고
[브랜치 이름 지정 안내](https://github.com/github/spec-kit/blob/main/CONTRIBUTING.md#branch-naming)에
따라 기능 브랜치에서 작업하세요. 그 체크아웃의 저장소 루트에서 다음 설정을
실행합니다.

macOS 또는 Linux:

```bash
uv sync --extra test
uv pip install --python .venv/bin/python -e .
.venv/bin/specify --help
```

Windows PowerShell:

```powershell
uv sync --extra test
uv pip install --python .venv\Scripts\python.exe -e .
.\.venv\Scripts\specify.exe --help
```

`uv sync --extra test`는 프로젝트 환경을 만들고 테스트 의존성을 설치합니다.
명시적인 편집 가능 설치는 그 같은 환경을 대상으로 합니다. Python 소스 편집에는
재설치가 필요 없습니다. 의존성 메타데이터가 바뀌면 설정을 다시 실행하세요.

## 2. 초점을 맞춘 자동 검사 실행

변경을 다루는 테스트부터 시작하되, 아래 예시 대신 실제 테스트 경로를 사용하세요.

macOS 또는 Linux:

```bash
.venv/bin/python -m pytest tests/test_agent_config_consistency.py -q
```

Windows PowerShell:

```powershell
.\.venv\Scripts\python.exe -m pytest tests/test_agent_config_consistency.py -q
```

전체 스위트를 실행하려면 테스트 파일을 `tests`로 바꾸세요. macOS에서는 Bash의
유니코드 처리가 UTF-8 로캘을 사용하도록 테스트에 `LC_ALL=en_US.UTF-8`을
설정하세요.

```bash
LC_ALL=en_US.UTF-8 .venv/bin/python -m pytest tests -q
```

CI와 같은 Ruff 버전을 실행합니다.

```bash
uvx ruff@0.15.0 check src tests
```

Ruff는 격리된 린트 도구로 실행됩니다. Spec Kit CLI와 pytest는 체크아웃의
`.venv`를 통해서만 실행하세요. 긍정 및 부정 커버리지, 회귀 근거, Windows
PowerShell 5.1 호환성, 추가 검사에 대해서는
[기여자 테스트 안내](https://github.com/github/spec-kit/blob/main/CONTRIBUTING.md#testing-deterministic-behavior)를
참고하세요.

## 3. 별도 프로젝트에서 CLI 테스트

체크아웃의 CLI를 사용해 Spec Kit 체크아웃 밖에 일회용 프로젝트를 초기화하세요.
에이전트의 [통합 키](../integrations/index.md)를 고르세요.

macOS 또는 Linux에서, Spec Kit 저장소 루트에서:

```bash
.venv/bin/specify init /tmp/spec-kit-test --integration <key> --script sh
```

Windows PowerShell에서, Spec Kit 저장소 루트에서:

```powershell
.\.venv\Scripts\specify.exe init "$env:TEMP\spec-kit-test" --integration <key> --script ps
```

독립적인 테스트마다 사용하지 않는 프로젝트 디렉터리를 고르세요. Python 스크립트
변형을 테스트할 때는 `--script py`를 사용하세요. 슬래시 명령의 동작이 바뀌는
경우에는 생성된 프로젝트를 코딩 에이전트에서 열고
[수동 테스트 절차](https://github.com/github/spec-kit/blob/main/CONTRIBUTING.md#manual-testing-process)를
따르세요.

생성된 프로젝트에서 CLI 명령을 더 실행한다면 Spec Kit 체크아웃 실행 파일의 절대
경로를 사용하세요. 전역 설치된 `specify`로 바꾸지 마세요. 아래 예시의 체크아웃
경로를 실제 체크아웃 경로로 바꾸세요. 예를 들어 생성된 프로젝트에서 CLI를 확인하고
git 확장을 추가하려면 다음과 같이 합니다.

macOS 또는 Linux:

```bash
"/path/to/spec-kit/.venv/bin/specify" --help
"/path/to/spec-kit/.venv/bin/specify" extension add git
```

Windows PowerShell:

```powershell
& "C:\path\to\spec-kit\.venv\Scripts\specify.exe" --help
& "C:\path\to\spec-kit\.venv\Scripts\specify.exe" extension add git
```

PowerShell 호출 연산자(`&`)는 공백이 포함된 경로를 포함해 따옴표로 묶인 경로를
실행합니다.

생성된 스크립트는 `.specify/scripts/` 아래에 있습니다. macOS나 Linux에서 Bash
스크립트 권한을 테스트할 때는 생성된 프로젝트에서 확인하세요.

```bash
ls -l .specify/scripts/bash/*.sh
```

스크립트에는 소유자 실행 비트가 설정되어 있어야 합니다. Windows의 PowerShell
스크립트에는 `chmod`가 필요하지 않습니다.

## 4. 해당하는 경우 패키징 확인

패키징 변경의 경우 Spec Kit 저장소 루트에서 휠을 빌드하세요.

```bash
uv build
```

`dist/`의 아티팩트를 확인하세요. 패키징 검증은 편집 가능 설치 개발 루프를 보완할
뿐, 테스트 환경을 대체하지 않습니다.

## 흔한 문제

| 증상 | 조치 |
| --- | --- |
| Python 의존성 누락 | 이 체크아웃의 루트에서 설정 명령을 다시 실행하세요. |
| 테스트나 CLI가 다른 체크아웃을 사용함 | 이 체크아웃의 명시적인 `.venv` 실행 파일 경로를 사용하세요. |
| 생성된 프로젝트에서 Git 명령을 사용할 수 없음 | 생성된 프로젝트에서 절대 `.venv` 경로를 사용해 체크아웃의 CLI로 `extension add git`을 실행하세요. |
| 잘못된 스크립트 변형이 생성됨 | `--script sh`, `--script ps`, `--script py`를 명시적으로 전달하세요. |
| 기업 네트워크에서 TLS 오류 | 예를 들어 `SSL_CERT_FILE`, `HTTPS_PROXY`, `HTTP_PROXY`로 인증서 저장소나 프록시를 설정하세요. 더 이상 사용되지 않는 `--skip-tls` 플래그는 아무 동작도 하지 않습니다. |

## 함께 보기

- [기여자 가이드](https://github.com/github/spec-kit/blob/main/CONTRIBUTING.md)
- [통합 기여하기](../integrations/contributing.md)
- [통합 API 레퍼런스](../integrations/api.md)
- [스펙 주도 개발 빠른 시작](../quickstart.md)
- [Spec Kit은 Spec Kit을 사용하나요?](https://github.com/github/spec-kit/blob/main/CONTRIBUTING.md#does-spec-kit-use-spec-kit)
