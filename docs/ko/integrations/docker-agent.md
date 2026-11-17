# Docker Agent

설정하기 전에 [Docker Agent](https://docs.docker.com/ai/docker-agent/)를 설치하세요.
통합 키는 `docker-agent`입니다. Spec Kit은 독립 실행형 `docker-agent` 바이너리나
Docker CLI 플러그인(`docker agent`) 중 어느 쪽이든 감지합니다.

## 다중 설치 안전성

이 통합은 스킬 디렉터리를 다른 통합과 공유하므로 다중 설치 안전으로 선언되어 있지
않습니다.
[공통 다중 설치 규칙](cli.md#can-i-install-multiple-integrations-in-the-same-project)을 참고하세요.

## 설정

```bash
specify init my-project --integration docker-agent
```

기존 Spec Kit 프로젝트에서는 `specify integration install docker-agent`를 사용하세요.
활성화, 업그레이드, 제거는 [통합 관리](cli.md)를 참고하세요.

## 설치되는 스킬과 에이전트 설정

스킬은 Codex와 Zed가 사용하는 것과 같은 디렉터리인
`.agents/skills/speckit-<command>/SKILL.md`로 설치됩니다. 선택한 에이전트 YAML에서
`skills: true`로 로컬 스킬을 활성화하고 파일시스템 읽기 접근을 제공하세요.
에이전트의 채팅에서 `/speckit-specify`, 또는 설치된 bug 확장의
`/speckit-bug-assess`로 스킬을 호출하세요.

선택적인 `--integration-options="--skills"` 설정 플래그는 기본값을 다시 명시할
뿐입니다. 이 통합은 항상 스킬을 스캐폴딩합니다. Spec Kit은 선택한 에이전트 YAML을
편집하거나 그 훅을 Spec Kit의 네이티브 이벤트 시스템으로 노출하지 않습니다.

## 워크플로 설정

워크플로 디스패치에는 에이전트 설정 참조가 필요합니다. 명령 스텝의 구조화된 런타임
필드를 우선하세요.

```yaml
- id: specify
  type: command
  command: speckit.specify
  integration: docker-agent
  integration_args:
    - ./agent.yaml
  integration_options:
    agent: root
    safety: balanced
  input:
    args: "Describe the feature"
```

`integration_args`는 비어 있지 않은 참조를 최대 하나 받으며, `-`로 시작해서는 안
됩니다. Docker Agent가 참조된 로컬 파일이나 레지스트리 항목을 검증합니다.
지원되는 `integration_options`는 `agent`와 `safety`입니다. safety는 `strict`,
`balanced`, `restricted`, `autonomous` 중 하나여야 합니다. 모델은
`integration_options.model`이 아니라 스텝의 최상위 `model`로 설정하세요. 알 수
없거나 형식이 잘못된 런타임 옵션은 명시적으로 실패합니다.

레거시 `SPECKIT_INTEGRATION_DOCKER_AGENT_EXTRA_ARGS=./agent.yaml` 환경 변수는
스텝별 참조가 제공되지 않을 때 계속 지원됩니다. 스텝별 참조는 포함된 플래그를
포함해 레거시 값 전체를 재정의합니다.
디스패치는 `run --exec`를 사용하고, 요청된 `--json` 및 `--model` 플래그를 추가하며,
프롬프트 텍스트가 CLI 옵션이 될 수 없도록 프롬프트를 `--` 뒤에 둡니다.
[스텝별 통합 설정](../workflow-steps/types.md#per-step-integration-configuration)을
참고하세요.
