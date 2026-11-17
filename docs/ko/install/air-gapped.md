# 엔터프라이즈 / 에어갭 설치

환경에서 PyPI나 GitHub 접근이 차단되어 있다면, 연결된 머신에서 휴대 가능한 휠 번들을 만들어 에어갭 대상으로 옮길 수 있습니다.

## 1단계: 연결된 머신에서 휠 빌드

> **중요:** `pip download`는 플랫폼별 휠을 해석합니다(예: PyYAML은 네이티브 확장을 포함합니다). 이 단계는 에어갭 대상과 **같은 OS 및 Python 버전**을 가진 머신에서 실행해야 합니다. 여러 플랫폼을 지원해야 한다면 각 대상 OS(Linux, macOS, Windows)와 Python 버전에서 이 단계를 반복하세요.

```bash
# 저장소 클론
git clone https://github.com/github/spec-kit.git
cd spec-kit

# 휠 빌드
pip install build
python -m build --wheel --outdir dist/

# 휠과 모든 런타임 의존성 다운로드
pip download -d dist/ dist/specify_cli-*.whl
```

## 2단계: `dist/` 디렉터리 옮기기

`dist/` 디렉터리 전체(`specify-cli` 휠과 모든 의존성 휠 포함)를 USB, 네트워크 공유, 또는 그 밖의 승인된 전송 방법으로 대상 머신에 복사합니다.

## 3단계: 에어갭 머신에 설치

```bash
pip install --no-index --find-links=./dist specify-cli
```

## 4단계: 프로젝트 초기화

네트워크 접근이 필요하지 않습니다. 기본적으로 번들된 자산이 사용됩니다.

```bash
specify init my-project --integration copilot
```

> **참고:** Python 3.11+가 필요합니다.
>
> **Windows 참고:** PowerShell 스크립트는 Windows에 포함된 Windows PowerShell 5.1
> (`powershell.exe`)을 대상으로 합니다. 오프라인 스캐폴딩에 PowerShell 7(`pwsh`)은
> 필요하지 않습니다. PowerShell 스크립트를 명시적으로 선택하려면 `--script ps`를
> 전달하세요. Windows의 비대화형 초기화는 기본적으로 이를 선택합니다.

## 컴포넌트 카탈로그와 소스

내장 통합과 번들된 컴포넌트는 네트워크 접근 없이도 계속 사용할 수 있습니다. 외부
통합 어댑터, 확장, 프리셋, 워크플로, 워크플로 스텝, 번들을 카탈로그 기반으로
설치하려면 카탈로그 메타데이터와 참조된 릴리스 아티팩트가 모두 필요합니다. 직접
또는 로컬 소스를 지원하는 컴포넌트 유형에서는 그 설치 경로에 카탈로그 메타데이터가
필요하지 않습니다.

승인된 내부 서비스에 접근할 수 있는 제한된 네트워크라면, 검토한 카탈로그와
아티팩트를 내부 HTTPS 엔드포인트에 호스팅한 다음 각 컴포넌트별 소스를 해당
`catalog add` 명령으로 등록하세요. 완전히 단절된 네트워크라면, 접근할 수 없는 원격
카탈로그에 의존하지 말고 검토한 컴포넌트 아티팩트를 승인된 절차로 옮긴 뒤 지원되는
직접 또는 로컬 소스로부터 설치하세요.

외부 통합 어댑터는 예외입니다. `specify integration install`에는 직접 또는 로컬
소스 옵션이 없습니다. 설치 허용 통합 카탈로그를 통해 어댑터를 해석하고 그
아카이브를 다운로드해야 합니다. 격리된 환경 안에서 외부 어댑터를 추가하려면 승인된
내부 카탈로그와 그 어댑터 아카이브를 모두 그곳에서 접근 가능하게 유지하거나,
대상의 연결을 끊기 전에 어댑터를 설치하세요. 아카이브만 옮기는 것으로는 충분하지
않습니다.

번들은 별도로 취급하세요. 옮겨진 번들 디렉터리나 ZIP은 번들 매니페스트를 제공할
뿐, 그것이 지명하는 컴포넌트 페이로드를 제공하지 않습니다. 연결을 끊기 전에
해당하는 비번들 컴포넌트를 따로 설치하세요. 명시적인 카탈로그 `source`를 가진
컴포넌트는 이미 설치되어 있어도 여전히 네트워크 검증이 필요하고, 워크플로 스텝
페이로드는 현재 스텝 카탈로그를 통해서만 해석됩니다. `specify bundle install --offline`에
의존하기 전에 [번들 오프라인 한계](../bundles/cli.md#offline-limits)를 검토하세요.

카탈로그 유형, 신뢰 정책, 소스 우선순위는 [카탈로그 레퍼런스](../overview/catalogs.md)를
참고하세요. 내부 엔드포인트에 자격 증명이 필요하다면
[인증](../reference/authentication.md)도 설정하세요.

<a id="git-credential-manager-on-linux"></a>

## Linux의 Git Credential Manager

Linux에서 Git 인증에 문제가 있다면 Git Credential Manager를 설치할 수 있습니다.

```bash
#!/usr/bin/env bash
set -e
echo "Downloading Git Credential Manager v2.6.1..."
wget https://github.com/git-ecosystem/git-credential-manager/releases/download/v2.6.1/gcm-linux_amd64.2.6.1.deb
echo "Installing Git Credential Manager..."
sudo dpkg -i gcm-linux_amd64.2.6.1.deb
echo "Configuring Git to use GCM..."
git config --global credential.helper manager
echo "Cleaning up..."
rm gcm-linux_amd64.2.6.1.deb
```
