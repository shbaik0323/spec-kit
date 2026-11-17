# PyPI에서 설치하기

Spec Kit은 Spec Kit 메인테이너가 유지 관리하는 [`specify-cli`](https://pypi.org/project/specify-cli/)로 PyPI에 게시됩니다. PyPI에서 설치하는 것은 [GitHub 소스](../installation.md#install-from-source--persistent-installation-recommended)에서 설치하는 것과 나란히 지원되는 두 번째 설치 경로입니다. 워크플로에 맞는 쪽을 사용하세요. 둘 다 같은 `specify` CLI를 제공합니다.

> [!NOTE]
> PyPI 릴리스 버전은 GitHub 릴리스 태그를 따릅니다(예: PyPI `0.12.11`은 `v0.12.11` 태그에 해당). `specify version`은 로컬 버전/런타임 점검일 뿐입니다. 설치된 버전은 보고하지만 `specify` 실행 파일이 어디서 왔는지는 보고하지 않으므로 PyPI 설치와 Git 설치를 구분할 수 없습니다. 설치 소스를 확인하려면 패키지 관리자가 기록하는 소스 메타데이터를 살펴보세요. `pipx list --json`은 각 도구의 정확한 설치 명세를 보고하고, uv/pip 설치에서는 패키지의 `*.dist-info` 디렉터리 안에 있는 [PEP 610](https://peps.python.org/pep-0610/) `direct_url.json`을 확인할 수 있습니다(Git 또는 URL 설치는 그곳에 저장소/아카이브 URL을 기록하고, 일반 PyPI 인덱스 설치는 그 파일을 만들지 않습니다). `pip show specify-cli`는 패키지 메타데이터만 출력하며 호스트 인터프리터에서 uv/pipx가 관리하는 환경을 볼 수 없다는 점에 유의하세요.

## Specify CLI 설치

이미 가지고 있는 Python 도구를 사용하세요.

```bash
# uv 사용(권장)
uv tool install specify-cli

# 또는 pipx 사용
pipx install specify-cli

# 또는 pip 사용
pip install specify-cli
```

Spec Kit을 uv로 설치했다면 `uv tool install specify-cli`를 다시 실행해도 설치된
버전이 유지될 뿐 업그레이드되지 않습니다. `specify version`이 보여 주는 버전을
[PyPI의 최신 버전](https://pypi.org/project/specify-cli/)과 비교하세요.
더 오래되었다면 `specify init`을 실행하기 전에, 원래 설치에 사용한 것과 같은
도구(uv, pipx, pip)로 아래의 [업그레이드](#upgrade)를 따르세요.
업그레이드만을 위해 설치 도구를 바꾸지 마세요. 예전 `specify`가 `PATH`에서 여전히
먼저 잡히는 채로 두 번째 설치가 생길 수 있습니다.

### 특정 릴리스 설치

재현 가능한 설치를 위해 정확한 버전을 고정하세요(사용 가능한 버전은 [PyPI](https://pypi.org/project/specify-cli/#history)나 [Releases](https://github.com/github/spec-kit/releases)에서 확인).

```bash
# uv 사용
uv tool install specify-cli==0.12.11

# 또는 pipx 사용
pipx install specify-cli==0.12.11

# 또는 pip 사용
pip install specify-cli==0.12.11
```

<a id="install-from-a-custom-or-private-package-index"></a>

## 사용자 정의 또는 비공개 패키지 인덱스에서 설치

일부 환경(기업 네트워크, 미러, 프록시, 아티팩트 피드)에서는 기본 공개 PyPI가 아닌 패키지 인덱스에서 `specify-cli`를 설치해야 합니다. 각 Python 도구는 다른 인덱스를 가리키는 방법을 제공합니다. 위의 설치 명령을 실행하기 전에 설정하세요. 여기에 표시된 자리 표시자를 여러분의 인덱스 URL로 바꾸세요.

```bash
# uv — 환경 변수 사용(명령 전체에 적용)
UV_DEFAULT_INDEX=https://your-index.example.com/pypi/simple/ uv tool install specify-cli

# uv — 플래그 사용
uv tool install --default-index https://your-index.example.com/pypi/simple/ specify-cli

# pipx — pip 인자를 그대로 전달
pipx install specify-cli --index-url https://your-index.example.com/pypi/simple/

# pip
pip install specify-cli --index-url https://your-index.example.com/pypi/simple/
```

> [!NOTE]
> 같은 인덱스 설정이 고정 설치, 업그레이드(`--force`/`--upgrade`), 일회성 사용에도 적용됩니다. 그 명령들에도 환경 변수나 플래그를 설정하세요. 인덱스에 인증이 필요하다면 도구의 문서를 따르고 자격 증명 환경 변수, keyring, netrc를 우선하세요. 셸 이력, 프로세스 목록, 로그를 통해 유출될 수 있으므로 명령줄 URL에 시크릿을 넣지 마세요. 시크릿을 커밋하지 마세요. 완전한 오프라인 설치는 [에어갭 설치 가이드](air-gapped.md)를 참고하세요.

## 확인

```bash
specify version
```

## 프로젝트 초기화

```bash
specify init <PROJECT_NAME> --integration copilot
```

<a id="upgrade"></a>

## 업그레이드

원래 설치에 사용한 것과 같은 도구로 패키지를 다시 설치해 업그레이드하세요. 원래 버전을 고정했다면 `uv tool upgrade`는 그 고정을 유지한다는 점에 유의하세요. 최신 PyPI 릴리스로 옮기려면 기존 버전 고정이 유지되지 않도록 고정 없는 설치 명령을 사용하세요.

```bash
# uv 사용
uv tool install --force specify-cli

# 또는 pipx 사용
pipx install --force specify-cli

# 또는 pip 사용
pip install --upgrade specify-cli
```

> [!NOTE]
> `specify self upgrade`는 현재 PyPI 기반 설치를 유지하지 않고 GitHub 소스 릴리스 URL로부터 `uv tool` 및 `pipx` 설치를 다시 빌드합니다. PyPI 경로에 머물고 싶다면 위의 패키지 관리자 명령을 사용하세요. 일반 `pip install specify-cli`는 관리되지 않는 설치로 취급됩니다. `pip install --upgrade specify-cli`로 업그레이드하세요. 자세한 내용은 [업그레이드 가이드](../upgrade.md)를 참고하세요.

## 제거

```bash
# uv 사용
uv tool uninstall specify-cli

# 또는 pipx 사용
pipx uninstall specify-cli

# 또는 pip 사용
pip uninstall specify-cli
```

## 다음 단계

[빠른 시작](../quickstart.md)으로 가서 첫 프로젝트를 초기화하세요.
