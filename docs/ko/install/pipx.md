# pipx로 설치하기

[pipx](https://pipx.pypa.io/)는 Python CLI 애플리케이션을 격리된 환경에 설치하는 도구입니다. [uv](https://docs.astral.sh/uv/)가 필요하지 않습니다.

## Specify CLI 설치

안정성을 위해 특정 릴리스 태그에 고정하세요(최신 태그는 [Releases](https://github.com/github/spec-kit/releases)에서 확인).

```bash
# 특정 안정 릴리스 설치(권장 — vX.Y.Z를 최신 태그로 교체.
# 앞의 v를 유지. 예: 0.12.11이 아니라 v0.12.11)
pipx install git+https://github.com/github/spec-kit.git@vX.Y.Z

# 또는 main의 최신 버전 설치(릴리스되지 않은 변경이 포함될 수 있음)
pipx install git+https://github.com/github/spec-kit.git
```

## 확인

```bash
specify version
```

## 업그레이드

```bash
pipx install --force git+https://github.com/github/spec-kit.git@vX.Y.Z
```

## 제거

```bash
pipx uninstall specify-cli
```

## 다음 단계

[빠른 시작](../quickstart.md)으로 가서 첫 프로젝트를 초기화하세요.
