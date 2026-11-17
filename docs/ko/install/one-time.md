# 일회성 사용 (uvx)

Spec Kit을 영구적으로 설치하지 않고 사용해 보고 싶다면 `uvx`로 격리된 환경에서 실행하세요. uv는 이 일회용 환경을 재사용을 위해 캐시하지만, PATH에 영구적인 `specify` 명령을 설치하지는 않습니다.

> [!NOTE]
> 아래 명령에는 **[uv](https://docs.astral.sh/uv/)**가 필요합니다. `command not found: uvx`가 표시되면 [먼저 uv를 설치하세요](uv.md).

## Specify CLI 실행

```bash
# 새 프로젝트 만들기(main의 최신 버전)
uvx --from git+https://github.com/github/spec-kit.git specify init <PROJECT_NAME>

# 또는 특정 릴리스 지정(vX.Y.Z를 Releases의 태그로 교체.
# 앞의 v를 유지. 예: 0.12.11이 아니라 v0.12.11)
uvx --from git+https://github.com/github/spec-kit.git@vX.Y.Z specify init <PROJECT_NAME>

# 현재 디렉터리에서 초기화
uvx --from git+https://github.com/github/spec-kit.git specify init . --integration copilot

# 또는 --here 플래그 사용
uvx --from git+https://github.com/github/spec-kit.git specify init --here --integration copilot
```

## 영구 설치를 대신 사용해야 하는 경우

Spec Kit을 정기적으로 사용할 계획이라면 영구 설치를 권장합니다.

- 도구가 설치된 상태로 유지되고 PATH에서 사용 가능
- uv의 일회용 캐시와 분리된 영구 도구 환경
- `uv tool list`, `uv tool upgrade`, `uv tool uninstall`을 통한 더 나은 도구 관리

영구 설치 지침은 메인 [설치 가이드](../installation.md)를 참고하세요.
