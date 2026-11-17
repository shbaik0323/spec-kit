# 통합 기여하기

이 가이드는 내장 통합, 외부 어댑터 개발, 커뮤니티 카탈로그 제출을 다룹니다.

지원되는 어댑터 계약, 렌더링 훅, 런타임 디스패치, 파일 소유권은
[통합 API 레퍼런스](api.md)부터 시작하세요.

## 내장 통합 추가하기

내장 통합은 Spec Kit 코어 팀이 유지 관리하며 CLI와 함께 제공됩니다.

체크아웃 설정은 [로컬 개발](../how-to/local-development.md)을 참고하세요.
[기여자 테스트 안내](https://github.com/github/spec-kit/blob/main/CONTRIBUTING.md#testing-deterministic-behavior)와
[검증 흐름](https://github.com/github/spec-kit/blob/main/CONTRIBUTING.md#recommended-validation-flow)을
따른 다음
[통합 PR 요구 사항](https://github.com/github/spec-kit/blob/main/CONTRIBUTING.md#integration-pull-requests)을
따르세요.

[공개 API](api.md)에 맞춰 어댑터를 구현하고, 긍정 및 부정 테스트를 추가하고,
에이전트별 설정, 호출, 제한 사항을 `docs/integrations/<key>.md`에 문서화하세요.
그 페이지는 탐색 드롭다운이 아니라 [통합 인덱스](index.md)에서 링크하고, 아래의
카탈로그 스키마를 사용해 탐색 메타데이터를 추가하세요.

내장 소스 등록과 비공개 구현 계약의 변경은 저장소의
[기여자 안내](https://github.com/github/spec-kit/blob/main/CONTRIBUTING.md#integration-pull-requests)를
따릅니다. 외부 어댑터를 설치하는 데는 필요하지 않습니다.

내장 항목 형식은 [카탈로그 스키마](catalog.md#catalog-schema)를 사용하세요.

## 외부 어댑터 개발하기

[외부 어댑터 계약](api.md#package-contract)을 사용해 어댑터를 구현하고
패키징하세요.
[디스크립터 레퍼런스](catalog.md#integration-descriptor-integrationyml)는 패키지
메타데이터를, [카탈로그 스키마](catalog.md#catalog-schema)는 게시되는 항목을
문서화합니다.

로컬 설치와 신뢰 결정은
[카탈로그로 설치하는 외부 어댑터](cli.md#catalog-installed-external-adapters)를
따르세요. 업데이트 동작은
[통합 업그레이드](cli.md#upgrade-an-integration)를 참고하세요.

### 어댑터 검증하기

공개 경로를 통해 패키지를 테스트하세요. 로컬 루프백 테스트 카탈로그를 등록하고,
그 중립 어댑터를 설치하고, 새 CLI 프로세스를 시작하고, 확장/프리셋 기여를 등록하고,
무해한 프로세스 대역(double)으로 명령/프롬프트 워크플로 디스패치를 실행해 보세요.

다음을 포함해
[라이프사이클 보장](api.md#external-adapter-lifecycle-guarantees)과
[파일 소유권 요구 사항](api.md#setup-and-file-ownership)을 다루세요.

- 유효하지 않은 메타데이터/클래스, 신뢰 거부, 안전하지 않은 아카이브, 설정 오류,
  업그레이드/제거 실패.
- 생성된 에이전트 파일과 분리되어 유지되는 패키지 코드. 수정된 파일, 관련 없는
  사용자 파일, 독립적인 워크플로 진행 상황, 동시 편집을 보존하는 롤백과 정리.
- 임포트 부수 효과가 없는 메타데이터 전용 명령. 로컬 소유권 증명을 사용한 손상된
  패키지의 강제 복구, 그리고 그 증명이 없을 때의 수동 정리 경고.
- 심볼릭 링크된 쓰기 대상, 예약된 루트, 안전하지 않은 레거시 경로, 기본 출력
  경로와 레거시 출력 경로 사이의 대소문자 무시 중복.
- 올바른 어댑터를 계속 사용하는 패키지 상대 임포트를 포함한, 프로젝트 간 동시
  디스패치.
- 이벤트를 지원하는 어댑터의 경우, 설치된 코드를 로드할 수 없을 때의 명시적인
  오류를 포함해 새 CLI 프로세스에서의 이벤트 전용 확장 추가/제거 및
  활성화/비활성화.

## 커뮤니티 카탈로그에 제출하기

외부 개발자는 탐색을 위해 `integrations/catalog.community.json`에 통합을 등재할 수
있습니다. 카탈로그 메인테이너는 어댑터 구현이 아니라 등재 메타데이터를 검토하며,
코드는 사용자가 검증해야 합니다. 등재가 설치 허가를 부여하지는 않습니다.

### 사전 요구 사항

1. **동작하는 외부 어댑터** — 위에서 설명한 대로 패키징되고 검증된 것.
2. **공개 저장소** — GitHub 또는 유사한 곳에 호스팅.
3. **문서** — 사용 지침이 담긴 README.
4. **라이선스** — 오픈소스 라이선스 파일.

[카탈로그 스키마](catalog.md#catalog-schema)를 사용해 항목을 준비하세요.
메타데이터는 패키징된 디스크립터와 일치해야 합니다. 고정된 아카이브 URL을
게시하세요. 아카이브 SHA-256 다이제스트를 권장합니다.

### 제출

1. [spec-kit 저장소](https://github.com/github/spec-kit)를 **포크**합니다
2. `integrations/catalog.community.json`의 `integrations` 키 아래에 **항목을 추가**합니다.

   ```json
   {
     "schema_version": "1.0",
     "integrations": {
       "sample-agent": {
         "id": "sample-agent",
         "name": "Sample Agent",
         "version": "1.0.0",
         "description": "Adapter for Sample Agent",
         "author": "your-name",
         "repository": "https://github.com/your-name/speckit-sample-agent",
         "download_url": "https://github.com/your-name/speckit-sample-agent/releases/download/v1.0.0/sample-agent.zip",
         "tags": ["cli"]
       }
     }
   }
   ```

3. 다음을 포함해 **풀 리퀘스트를 엽니다**.
   - 카탈로그 항목
   - 통합 저장소 링크
   - `integration.yml`이 유효하다는 확인

### 버전 업데이트

카탈로그에서 통합 버전을 업데이트하려면 다음과 같이 합니다.

1. 통합의 새 버전을 릴리스합니다
2. 버전, 고정된 아카이브 URL, 다이제스트(제공한 경우)를 갱신하는 PR을 엽니다
3. 하위 호환성을 보장하거나 호환성을 깨는 변경을 문서화합니다

## 함께 보기

- [통합 API 레퍼런스](api.md)
- [커뮤니티 통합](../community/integrations.md)
- [통합 목록](index.md)
- [카탈로그 스키마](catalog.md)
