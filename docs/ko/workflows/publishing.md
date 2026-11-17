# 워크플로 게시하기

다른 프로젝트가 같은 정의와 동반 파일을 설치할 수 있도록 버전이 지정된 워크플로
패키지를 게시하세요. 워크플로 카탈로그 제출과 사용자 정의 워크플로 스텝 제출은
서로 다른 경로를 사용합니다. 둘을 서로 바꿔 쓸 수 있는 것으로 취급하지 마세요.

## 릴리스 준비

워크플로를 공유하기 전에 다음을 제공하세요.

- 고유한 ID, 설명적인 이름, 따옴표로 묶은 `X.Y.Z` 버전을 갖춘 유효한
  `workflow.yml`.
- 프로세스, 입력, 스텝 그래프, 필요한 통합, 도구, 자격 증명, 출력, 실패 동작, 부수
  효과를 설명하는 README.
- 오픈소스 라이선스와, 가급적 변경 로그.
- 관련 거부, 실패, 재개 경로를 포함해 실제 프로젝트에서 워크플로를 실행한 근거.

초점이 좁은 스텝, 설명적인 입력 프롬프트, 의미 있는 리뷰 게이트, 고정된 선택지에
대한 enum을 사용하세요. 파괴적인 작업, 자격 증명 접근, 네트워크 호출, 상승된
권한을 설명하세요. 선언된 `requires` 값은 기대 사항을 문서화할 뿐, 권한을
강제하거나 실행을 샌드박스화하지 않습니다.

패키지 구조와 완전한 번들 예시는 작성 가이드를 참고하세요.
릴리스 전에 로컬 정의를 테스트하세요.

```bash
specify workflow run ./workflow.yml --input spec="Build a kanban board"
specify workflow status <run_id>
specify workflow resume <run_id>
```

`run`은 워크플로를 검증한 다음 실행합니다. 드라이 런이 아닙니다. 변경을 일으킬 수
있는 예시는 일회용 프로젝트에서 테스트하세요.

## 패키징과 태그

`workflow.version`과 일치하는 버전 태그에서 게시하세요.

```bash
git tag v1.0.0
git push origin v1.0.0
```

단일 파일 워크플로라면 태그에 고정된 원시 URL로 충분합니다.

```text
https://raw.githubusercontent.com/your-org/my-workflow/v1.0.0/workflow.yml
```

정의에 스크립트나 다른 리소스가 필요하다면 패키지 전체를 `.zip`, `.tar.gz`,
`.tgz`로 게시하세요. 아카이브는 `workflow.yml`을 루트 또는 하나의 최상위 디렉터리
안에 담아야 합니다. 원시 YAML 다운로드에는 저장소의 인접 파일이 포함되지
않습니다. 패키지된 리소스는 `context.workflow_dir`이나 `SPECKIT_WORKFLOW_DIR`을
통해 참조하세요.

브랜치 URL이나 `releases/latest`가 아니라 고정된 릴리스 URL을 사용하세요. 특히
과거 릴리스를 공개할 때는 다운로드 가능한 콘텐츠의 SHA-256 다이제스트를 계산해
게시하세요.

## 게시된 아티팩트 테스트

릴리스 소스를 검토한 뒤, 초기화된 테스트 프로젝트에 게시된 정확한 아티팩트를
설치하세요.

```bash
specify workflow add my-workflow \
  --from https://raw.githubusercontent.com/your-org/my-workflow/v1.0.0/workflow.yml
specify workflow info my-workflow
specify workflow run my-workflow --input spec="Build a kanban board"
```

동반 파일이 있는 패키지라면 버전이 지정된 아카이브 URL로 바꾸세요.
설치만이 아니라 게이트와 재개도 실행해 보세요. `info`는 메타데이터, 입력, 스텝을
보여 줄 뿐, 모든 셸 명령을 표시하거나 안전성을 증명하지 않습니다.

## 워크플로 카탈로그 항목 제출

워크플로 항목은 현재
[`workflows/catalog.community.json`](https://github.com/github/spec-kit/blob/main/workflows/catalog.community.json)을
편집하는 풀 리퀘스트를 사용합니다. 워크플로 정의 제출용 이슈 양식은 없습니다.
별도의 Workflow Step Type Submission 양식은 실행 가능한 사용자 정의 스텝 패키지를
위한 것입니다.

저장소를 포크하고 `workflows` 매핑 아래에 워크플로 ID의 알파벳순으로 워크플로를
추가하세요. 예:

```json
{
  "schema_version": "1.0",
  "workflows": {
    "my-workflow": {
      "id": "my-workflow",
      "name": "My Workflow",
      "description": "A repeatable review process",
      "author": "Example Team",
      "version": "1.0.0",
      "url": "https://raw.githubusercontent.com/your-org/my-workflow/v1.0.0/workflow.yml",
      "sha256": "<64 hexadecimal digits for the published artifact>",
      "repository": "https://github.com/your-org/my-workflow",
      "license": "MIT",
      "requires": {
        "speckit_version": ">=1.0.0"
      },
      "tags": ["review", "automation"],
      "created_at": "2026-11-03T00:00:00Z",
      "updated_at": "2026-11-03T00:00:00Z"
    }
  }
}
```

예시 메타데이터, 호환성 요구 사항, URL, 다이제스트를 릴리스의 실제 값으로
바꾸세요. `requires`는 정의와 일관되게 유지하세요.
현재 릴리스는 최상위에 남습니다. 과거 버전에는 카탈로그 레퍼런스에 설명된 대로
`releases`를 사용하세요.

다음을 담아 초점이 좁은 카탈로그 PR을 제출하세요.

- 워크플로 ID, 이름, 버전, 저장소, 라이선스.
- 고정된 다운로드 URL과 다이제스트.
- README와 스텝 그래프 링크, 필요한 통합, 부수 효과.
- 게이트 및 재개 동작을 포함한 설치 및 종단 간 테스트 근거.

메인테이너는 카탈로그 항목의 형식과 완전성을 확인합니다. 이는 워크플로 정의,
스크립트, 의존성에 대한 **보안 검토가 아닙니다**. 카탈로그 수락은 보증도, 지원
약속도, 코드가 감사를 받았다는 증거도 아닙니다.
사용자는 설치하고 사용하기 전에 소스를 검증해야 합니다.

커뮤니티 카탈로그는 기본적으로 탐색 전용입니다. 수락되면 워크플로를
`specify workflow search`로 탐색할 수 있게 될 뿐, ID로 자동 설치할 수 있게 되지는
않습니다. 릴리스를 검증한 뒤 고정된 URL을 명시적으로 설치하거나, 신뢰하고 설치를
허용하도록 별도로 설정한 카탈로그를 사용하세요.

## 릴리스 업데이트

1. `workflow.version`, 문서, 변경 로그를 갱신합니다.
2. 해당하는 실패 및 재개 경로를 포함해 변경된 정의를 테스트합니다.
3. 새 태그와 버전이 지정된 아티팩트를 게시하고, 그 URL에서의 설치를 테스트합니다.
4. 현재 `version`, `url`, 다이제스트, 릴리스별 요구 사항을 갱신하는 카탈로그 PR을
   제출합니다. 공개된 과거 릴리스를 보존하세요.

워크플로, 사용자 정의 스텝, 오버레이, 참조된 스크립트를 업데이트할 때는 소스 검토를
다시 하세요. 패키지 설치와 다이제스트 검증은 어느 바이트가 설치되었는지를 확립할
뿐, 그 바이트가 안전한지는 확립하지 않습니다.

## 함께 보기

- [워크플로 개발하기](developing.md)
- [워크플로 카탈로그](catalogs.md)
- [설치 CLI](cli.md#install-a-workflow)
- [셸 실행 검토](../workflow-steps/types.md#reviewing-shell-execution)
- [표현식 보간과 셸 안전성](expressions.md#interpolation-and-shell-safety)
- [커뮤니티 워크플로](../community/workflows.md)
- [사용자 정의 워크플로 스텝 게시하기](../workflow-steps/publishing.md)
